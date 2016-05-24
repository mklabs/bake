# Bake

> Make like task runner, with npm script goodness

## The Gist

Given the following Makefile

```make
foo2:
	echo foo2

foo: prefoo
	echo foo

prefoo:
	echo prefoo

foobar: prefoobar
	echo foobar

prefoobar:
	echo blahblah

all: foo foo2 foobar
```

Run bake

    $ bake
    bake info Invoking foo target
    bake info Invoking prefoo target
    prefoo
    foo
    bake info Invoking foo2 target
    foo2
    bake info Invoking foobar target
    bake info Invoking prefoobar target
    blahblah
    foobar
    bake info ✔ Build sucess in 41ms

## Usage

    $ bake <target> [options]

    Options:
      -h, --help         Show this help
      -v, --version      Show package version
      -d, --debug        Enable extended output

    Targets:
      foo2               Run target foo2
      foo                Run target foo
      prefoo             Run target prefoo
      foobar             Run target foobar
      prefoobar          Run target prefoobar

## What is Bake ?

Bake is a little experiment to implement a simple task runner similar to
Make in JavaScript, while bringing in the conveniency of npm scripts with
`$PATH` and environment variables.

It takes a similar approach to Make with a very close syntax.

Recipes (or rules, the commands defined for a target / task), are executed with
`bash -c` (similar to npm scripts).

For now, basic variable and target declarations are supported, along
with basic prerequities support (eg. task depending on other tasks).

- No need to prefix rules with $@ for silent output
- Slightly easier variables substitution (eg. $VARIABLE instead of $(VARIABLE)
- Variable declarations
- No tab requirements
- Every binaries in `./node_modules/.bin` is made available, like npm does for npm scripts.
- Real bash scripting (variables / multiline)
  - Recipes are executed with `bash -c` instead of executing each rule, line by line like Make does.
  - Multiline echo support
  - Bash variable support

**todo**

- Environment variables `bake_*` similar to `npm_*` available in npm scirpts
- Variable substitution for prerequities and targets (right now, replacement is done only for rules / recipes)
- Implement [pattern rules](https://www.gnu.org/software/make/manual/html_node/Pattern-Intro.html#Pattern-Intro)
- Implement [automatic variables](https://www.gnu.org/software/make/manual/html_node/Automatic-Variables.html#Automatic-Variables)
- Implement mtime check (a target needs to be rebuilt if it does not exist of if it's older than any of the prerequities)

### Makefiles goodness

#### Bash scripting

```make
help:
  echo """
    Some help message here:
    Run with bake help
  """

all: help
```

This, with Make, would throw an error

    $ make help
    echo """
    /bin/sh: 1: Syntax error: Unterminated quoted string
    Makefile:8: recipe for target 'help' failed
    make: *** [help] Error 2

While, bake is ok with it

    $ bake help
    bake info Invoking help target

    Some help message here
    Run with bake help

    bake info ✔ Build sucess in 43ms

#### Make like variables

```make
somevar = anything after "=" is considered the value till the end of the line
OUT_FLAGS = output.js

build-js:
  cat a.min.js b.min.js > $OUT_FLAGS
  echo JS file built
```

The syntax and behavior is a bit different. Instead of using `$(var)` syntax,
`$var` is used instead (that might changed to allow bash variables within
recipes, which uses this syntax).

#### Task dependencies

Use prerequities to specify tasks that depends on other tasks.

**Makefile**

```
prebuild:
  echo done

build: prebuild

deploy: build
```

**Output**

    $ bake deploy
    bake info Invoking deploy target
    bake info Invoking build target
    bake info Invoking prebuild target
    done
    bake info ✔ Build sucess in 50ms

### npm like environment

Recipes run in an environment very similar to the environment npm scripts are
run in, namely the `PATH` environment variable.

#### path

If you depend on modules that define executable scripts, like test suites,
then those executables will be added to the `PATH` for executing the scripts.

So, if your package.json has this:

```json
{
  "name" : "foo" ,
  "dependencies" : { "bar" : "0.1.x" }
}
```

then you could run bake to execute a target that uses the `bar` script, which
is exported into the `node_modules/.bin` directory on `npm install`.
