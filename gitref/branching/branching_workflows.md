---
layout: page
title: "Branching Workflows"
permalink: /gitref/branching/branching_workflows
---

## Topic branches workflow

* A topic branch is a short lived branch that you create and use for a single particular feature or related work.
* You create a topic branch for whatever task you are working on and delete it after you merge it into the master branch.
* This technique lets you context switch by separating your work into branches where all work in a branch is for the branch topic.

## Long running branch workflow

* Only stable code in the master branch.  Possibly just code that has been released.
* There is a parallel branch named develop or next that is used to test stability.
    * This branch is not necessarily always stable, but when it gets to stable state it can be merged into master.
    * This branch is used to pull in [topic branches]({% link gitref/definitions.md %}#topic-branch) and run them though testing.
* Some larger projects have a proposed or pu (proposed updates) branch that has integrated branches which may not be ready to go into the next or master branch yet.
    * The idea is that your branches have various levels of stability.  When they reach a stable level, they are merged into the branch above them.
* The multiple long running branches approach is helpful when you have very large or complex projects.