---
layout: page
title: "Git Reference"
permalink: /gitref/
---

[comment]: <> (TODO: Good idea to add table of contents at top of each page so you can jump to sections.)
[comment]: <> (TODO: Fix some of the metadata around the site and see if you can update th styling to have a breadcrumb or back at top instead of "My References".)

## Workflow

[comment]: <> (TODO: Once you build out the content may want to sue the subgraph option to make more granular pages.)

{% mermaid %}
 flowchart LR
    init([Create or clone repo])
    branching([Brach based development])
    work([Work on code locally])
    history([View repository history])
    commit([Commit your changes])
    remotes([Remote repositories])
    

    click init "{% link gitref/init.md %}"
    click work "{% link gitref/work.md %}"
    click branching "{% link gitref/branching/branching.md %}"
    click history "{% link gitref/repo_history/repo_history.md %}"
    click commit "{% link gitref/commit.md %}"
    click remotes "{% link gitref/remotes.md%}"


    init-->branching
    init-->history
    branching-->work
    history<-->work
    work-->commit
    commit-->remotes
    remotes-->init
    
{% endmermaid %}

[comment]: <> (TODO: May want to add some quick links so that someone who does not know which mermaid bubble to look to can jump to info.)

## Configuration
[comment]: <> (TODO: Need to fill out this section.)