# Aka Midori

Aka Midori (赤緑, "red-green") is a GitHub Action that holds pull requests
to test-driven development. A pull request may have:

- **one commit**, or
- **two commits**: the first adds a failing test (red), the second makes it
  pass (green).

Anything else fails the check.

## How it works

On a pull request with two commits, Aka Midori checks out the first one,
installs dependencies, and runs your tests. If they pass, the check fails:
the first commit was supposed to add a test that does not pass yet. A pull
request with three or more commits fails straight away.

Aka Midori does not run the tests on the last commit. Your regular test
workflow already does that, and together the two checks reject two green
commits, two red ones, and long histories.

Merge commits count, so authors rebase onto the base branch instead of
merging it in.

## Usage

Set up your toolchain, then call the action with the commands that install
dependencies and run the tests:

```yaml
name: tdd

on:
  pull_request:

permissions:
  contents: read

jobs:
  tdd:
    runs-on: ubuntu-latest
    timeout-minutes: 20
    steps:
      - uses: actions/checkout@v7
      - uses: astral-sh/setup-uv@v10.3.0
      - uses: lymagics/aka-midori@v1
        with:
          install: uv sync --locked
          test: make unit
```

To block merging, mark the `tdd` job as a required status check in branch
protection.

## Inputs

| Input     | Required | Default                    | Meaning                                                        |
|-----------|----------|----------------------------|----------------------------------------------------------------|
| `test`    | yes      |                            | Runs the tests; the first of two commits must make it fail.    |
| `install` | no       | none                       | Installs dependencies before the tests; it must succeed.       |
| `base`    | no       | the pull request base SHA  | Commit the pull request starts from.                           |
| `head`    | no       | the pull request head SHA  | Last commit of the pull request.                               |

A failing `install` fails the check, so a broken lockfile never passes for
a red test.

The action fetches full history itself when the checkout is shallow, and
puts the original checkout back when it is done, so later steps in the job
see the same tree.

## License

[MIT](LICENSE)
