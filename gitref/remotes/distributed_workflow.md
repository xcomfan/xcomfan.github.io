---
layout: page
title: "Distributed Work"
permalink: /gitref/distributed_work
---

## Summaries of common distributed Git workflows

### Centralized workflow

This is a basic pull, make changes and if you try to submit after someone else has submitted, you pull the changes down and merge before getting to upload them.

You can use git fetch origin to get the updates of the master branch then merge the master branch into your topic branch before submitting code.  This is better than merging into master and having your work stuck in master in your local repo.  Just remember to merge origin/master into your topic branch instead of your local master.

### Integration-Manager Workflow

This workflow has the following flow

1. The project maintainer pushes to their public repository.
2. A contributor clones that repository and makes changes.
3. The contributor pushes to their own public copy.
4. The contributor sends the maintainer an email asking them to pull changes.
5. The maintainer adds the contributor's repository as a remote and merges locally.
6. The maintainer pushes merged changes to the main repository.

The main advantage of this workflow is that you can continue to work and the maintainer can pull in your changes at any time.  Contributor's don't have to wait for the project to incorporate their changes.  Each party can work at their own pace.

### Dictator and lieutenants workflow

This workflow is generally used by huge projects such as the Linux kernel.

Various integration managers are in charge of certain parts of the repository.  They are called lieutenants.  All lieutenants have one integration manager known as the benevolent dictator.  The benevolent dictator pushes from their directory to a reference repository from which all the collaborators need to pull.  The process has the following flow.

1. Regular developers work on their topic branch and rebase their work on top of master.  The master branch is that of the reference repository to which the dictator pushes.
2. Lieutenants merge the developers topic branches into their master branch.
3. The dictator merges the lieutenants master branches into the dictator's master branch.
4. Finally, the dictator pushes that master branch to the reference repository so the other developers can rebase on it.

This workflow is not common but can be very useful in big projects or highly hierarchical environments.  It allows the project leader (the dictator) to delegate much of the work and collect large subsets of code at multiple points before integrating them.

### Git workflows references

[Patterns for Managing Source Code Branches by Martin Fowler](https://martinfowler.com/articles/branching-patterns.html)

## Best practices when committing to a project

[comment]: <> (TODO: Eventually I will break up this file and this should be in its own file so I can reference to it from other places.)

* Before you submit changes run `git diff --check` which will identify any whitespace changes in your change and list them for you.  Whitespace changes may annoy other developers.

* Try to make each commit a logically separate changeset.
    * For example, use your staging area to split your code into at least one issue per commit
    * If some changes modify the same file use `git add --patch` to partially stage files.
    [comment]: <> (TODO: Need a reference link above on how to use git patch once you have that written up.)

## GitHub workflow

[comment]: <> (TODO: Below should be broken out into its own file.)

### Forked Public Project

* First you clone the project you want to contribute to and make a feature branch that you will work on. 
* When you are ready to contribute the work in your feature branch to the maintainers go to the original project page and click the Fork button to create a fork of the project.
* Add the fork as a remote in your environment, and push your work to this new remote.
    * It is easiest to push the topic branch you are working on to your forked repository, rather than merging that work into your master branch and pushing that.  The reason is if your work is not selected or is  cherry picked, you don't have to rewind your master branch.
* Push your work `git push -u myfork featureA`
* Once your work has been pushed to your fork of the repository you need to notify the maintainers of the original project that you have work you would like them to merge by generating a pull request.  
    * You can generate the pull request via the project website
    * You can also use the git request-pull command and email the output
        * git request-pull command takes the base branch into which you want your topic branch pulled and the Git repository URL you want them to pull from, and produces a summary of all the changes you're asking to be pulled.
* On a project where you are not the maintainer its generally easier to have a branch like master always track origin master and to do your work in topic branches that you can easily discard if rejected.