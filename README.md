# dleske-tests - Generic test scriptage

For use in whatever projects.

Tests in this repository should require as little configuration as possible and should safely run whether or not there are testable candidates in a given workspace.  For example, the *shellcheck* script, which lint-checks shell scripts, should not return a non-zero value in the absence of shell scripts.

## Use in project repositories

This repository is intended to be used as a submodule in project repositories.  To add this repository as a submodule of `project-x`, use something like the following:

```ShellSession
$ cd <project-x-dir>
$ git submodule add <tests-repo.git>
```

This repository will now be cloned in the `tests/` subdirectory of the `project-x` working copy.  

Updated tests can be pulled to project repositories via `git pull` in the submodule directory, or more generically, from the project repo's root:

```ShellSession
$ git submodule foreach git pull origin master
Entering 'tests'
From https://git.computecanada.ca/dleske/tests
 * branch            master     -> FETCH_HEAD
Already up to date.
```

## Gitlab CI

This is just an example, but one project uses the following for CI (`.gitlab-ci.yml`):

```
image: alpine:latest

variables:
  PYLINTRC: tests/extras/pylintrc

before_script:
- apk update && apk add gcc musl-dev py3-pip python3-dev git bash
- pip3 install pylint
- git submodule sync
- git submodule update --init

stages:
- parse
- test

syntax:
  stage: parse
  script:
    - tests/test-all
```

## Test script guidelines

The following guidelines have been followed in this test suite:

* Test scripts should not fail if a dependency cannot be found but there is nothing to test.  In other words, CI for a pure Python project should not fail because `shellcheck` can't be found in the test environment.

    The `shellcheck` script, for example, if unable to find the `shellcheck` executable, will only *fail* (return non-zero exit status) if there are shell scripts to be tested.

* Test scripts should be "opt-in" if possible.  I'm not sure about this one, but introduction of the test suite should not cause reams of warnings to scroll off the user's terminal or CI window.  Test scripts flag scripts that look like candidates, but aren't tested, as being skipped.

    To opt in, a script would include the name of the script and potentially relevant directives in the script header, before any non-commented lines.  Without this a potential candidate is skipped.

## Scripts in use

### Shellcheck

Shellcheck looks for and evaluates the perceived correctness of shell scripts of the Bourne shell family.  It requires a `shellcheck` executable; if candidate scripts are found and no executable can be found the test will fail.

Shell scripts that are not explicitly identified as candidates are skipped.

### Pylint

Pylint runs a Python linter against candidate Python files.  It tends to be fairly demanding and so a serviceable RC file is included in `extras/`.
