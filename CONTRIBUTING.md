# Contributing

As of 2026-, we are not accepting contributions from outside the team.
If you want to create a pull request, please contact a maintainer first.

You can use your favorite Python version manager (asdf, pyenv, ...) as long
as it follows `.python-version`.

Install [prek](https://prek.j178.dev/) if it is not already installed.

Install pre-commit hooks in your git working copy:

    prek install --hook-type pre-commit --hook-type commit-msg

Use [Conventional Commits](https://www.conventionalcommits.org/).

Update documentation:

    pdm docs
