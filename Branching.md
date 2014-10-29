# Branching

We use several branches:

- The **master** branch is used for releases. Each commit on this branch should be a merge from the development branch and changing the version to final. Only one of the main developers should touch this branch.
- The **development** branch is used for latest snapshots. Each new feature will first be merged into this branch to be able to test if it works with the rest of the code. These should always be done through pull requests. Developers are discouraged from pushing to this branch directly.
- For each new feature of bugfix a new branch should be created. This branch should start with the issue number and then a short description (e.g. 19-fix-npe-connectionmanager). These branches should simply be pushed to github and then you can create a pull request to merge it back to development. Make sure that this can be done cleanly (when you are behind the current development version, you can always use *git rebase*).
- New features unrelated to a bugfix, should be put in a new branch. The name must not start with a number, but be a short description (e.g. persisting-connections). Same as above, merges should be possible to do cleanly.