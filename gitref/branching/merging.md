---
layout: page
title: "Merging Branches"
permalink: /gitref/branching/merging
---

Click on the nodes for details of each workflow step.

{% mermaid %}
 flowchart TD
    start([Begin merging two branches])
    abort([Abort merge])
    resolve([Resolve merge conflicts])
    undo([Undo a completed merge])
    finish([Merge commit created])
    
    start-->abort
    start-->resolve
    start-->|If branches are not diverged fast forward merge|finish
    abort-->start
    undo-->start
    resolve-->finish
    finish-->undo

    click start "{% link gitref/branching/starting_merge.md %}"
    click abort "{% link gitref/branching/abort_merge.md %}"
    click resolve "{% link gitref/branching/resolving_merge_conflicts.md %}"
    click undo "{% link gitref/branching/undoing_merge.md %}"

{% endmermaid %}

[comment]: <> (TODO: REV MARKER)

[comment]: <> (TODO: Need to write up an explainer on merging and link to that in the concepts section.)