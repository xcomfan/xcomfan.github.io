---
layout: page
title: "Git Reference"
permalink: /gitref/
---


[comment]: <> (TODO: Good idea to add table of contents at top of each page so you can jump to sections.)

## Workflow

[comment]: <> (TODO: Once you build out the content may want to sue the subgraph option to make more granular pages.)
[comment]: <> (TODO: Now that I am working my way through I think this chart needs a complete overhaul and most of the content is in a working locally sub-graph)
[comment]: <> (TODO: Once you are content complete may want to build a new grapsh with direct links to the sub pages and habe the full index below.)

{% mermaid %}
 flowchart LR
    subgraph work_local_sub [Work Locally]
        direction LR
        history([Work with repository history])
        branching([Create and work with branches])
        work([Work on your local branch])
        stashing([Stashing your work])
        commit([Committing your work])
    end

    init([Create or clone repo])
    
    remotes([Remote repositories])
    
    click init "{% link gitref/init.md %}"
    click history "{% link gitref/repo_history/repo_history.md %}"
    click branching "{% link gitref/branching/branching.md %}"
    click work "{% link gitref/work/work.md %}"
    click stashing "{% link gitref/stashing.md %}"
    click commit "{% link gitref/committing/commit.md %}"
    click remotes "{% link gitref/remotes/remotes.md%}"

    init-->work_local_sub
    work_local_sub-->remotes
{% endmermaid %}

## Contents

### Initializing your project

[Create or clone a repository]({% link gitref/init.md%})

### Work with repository history

### Working with branches

### Stashing your work

### Committing your work

### Working with remote repositories