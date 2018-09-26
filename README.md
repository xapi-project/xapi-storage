[![Build Status](https://travis-ci.org/xapi-project/xapi-storage.svg?branch=slate)](https://travis-ci.org/xapi-project/xapi-storage)

# Source of https://xapi-project.github.io/xapi-storage

This is the source code of the SMAPIv3 doc website
<https://xapi-project.github.io/xapi-storage>. The website is based on [Slate].

The files in `source/includes/` are auto-generated and uploaded by the Travis
builds of the main branch. The Travis build of this branch generates the
website and pushes it to the `gh-pages` branch.

# Updating Slate

Our custom modifications are on top of a given snapshot of Slate.
The version of Slate the website is based on can be updated by simply rebasing
this branch on top of the `master` branch from the `slate` repository.

[Slate]: https://github.com/lord/slate
