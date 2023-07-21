---
layout: page
title: "Merging Branches"
permalink: /gitref/branching/merging
---

Click on the nodes for details of each workflow step.

Dotted lines indicate optional path

{% mermaid %}
 flowchart TD
    start([Begin merging two branches])
    abort([Abort merge])
    resolve([Resolve merge conflicts])
    undo([Undo a completed merge])
    merge_commit[Merge commit created]
    finish[Merge completed]
    
    start-.->abort
    start-->|If branches are diverged need to resolve conflict|resolve
    start-->|If branches are not diverged fast forward merge|finish
    resolve-->merge_commit
    finish-.->undo
    merge_commit-->finish

    click start "{% link gitref/branching/starting_merge.md %}"
    click abort "{% link gitref/branching/abort_merge.md %}"
    click resolve "{% link gitref/branching/resolving_merge_conflicts.md %}"
    click undo "{% link gitref/branching/undoing_merge.md %}"

{% endmermaid %}

[comment]: <> (TODO: Need to write up an explainer on merging and link to that in the concepts section.)