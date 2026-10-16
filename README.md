# Aka Midori

Aka Midori (赤緑, "red-green") is a GitHub Action that holds pull requests
to test-driven development. A pull request may have:

- **one commit**, or
- **two commits**: the first adds a failing test (red), the second makes it
  pass (green).

Anything else fails the check.

## How it works

Aka Midori replaces the steps that install dependencies and run your tests.
It always runs them on the checked-out commit, so your test job keeps doing
what it did before. On a pull request it also checks the history:

- with one commit, nothing more happens;
- with two commits, it first checks out the first one, installs
  dependencies, and runs the tests. If they pass, the check fails: the first
  commit was supposed to add a test that does not pass yet;
- with three or more commits, the check fails straight away.

Outside a pull request (a push to `main`, a release) it only runs the tests.

The first commit is checked before the last one, so dependencies end up
installed for the last commit and later steps in the job see the same tree
as without the action.

Merge commits count, so authors rebase onto the base branch instead of
merging it in.

## Usage

Swap your install and test steps for the action:

```yaml
name: tests

on:
  push:
    branches: [main]
  pull_request:

jobs:
  tests:
    runs-on: ubuntu-latest
    timeout-minutes: 20
    steps:
      - uses: actions/checkout@v7
      - uses: astral-sh/setup-uv@v10.3.0
      - uses: lymagics/aka-midori@v0.0.2
        with:
          install: uv sync --locked
          test: make unit
```

In a matrix, every leg checks the first commit. That repeats the same
check, so keep the matrix small or accept the extra minutes.

## Inputs

| Input     | Required | Default                    | Meaning                                                        |
|-----------|----------|----------------------------|----------------------------------------------------------------|
| `test`    | yes      |                            | Runs the tests; the last commit must pass, the first of two must fail. |
| `install` | no       | none                       | Installs dependencies before each test run; it must succeed.   |
| `base`    | no       | the pull request base SHA  | Commit the pull request starts from.                           |
| `head`    | no       | the pull request head SHA  | Last commit of the pull request.                               |

A failing `install` fails the check, so a broken lockfile never passes for
a red test.

The action fetches full history itself when the checkout is shallow, and
puts the original checkout back when it is done, so later steps in the job
see the same tree.

## License

[MIT](LICENSE)
