---
layout: page
title: "Git Reference"
permalink: /gitref/
---

[comment]: <> (TODO: Fix some of the metadata around the site and see if you can update the styling to have a breadcrumb or back at top instead of MyReferences)
[comment]: <> (TODO: Good idea to add table of contents at top of each page so you can jump to sections.)
## Workflow

[comment]: <> (TODO: Once you build out the content may want to sue the subgraph option to make more granular pages.)

{% mermaid %}
 flowchart LR
    init([Create or clone repo])
    branching([Branching])
    work([Work locally])
    stashing([Stashing ])
    history([View repository history])
    commit([Committing])
    remotes([Remote repositories])
    
    click init "{% link gitref/init.md %}"
    click branching "{% link gitref/branching/branching.md %}"
    click work "{% link gitref/work.md %}"
    click stashing "{% link gitref/stashing.md %}"
    click history "{% link gitref/repo_history/repo_history.md %}"
    click commit "{% link gitref/commit.md %}"
    click remotes "{% link gitref/remotes.md%}"

    init-->branching
    init-->history
    branching-->work
    history<-->work
    work-->commit
    work-->stashing
    stashing-->commit
    commit-->remotes
    remotes-->init
    
{% endmermaid %}

[comment]: <> (TODO: May want to add some quick links so that someone who does not know which mermaid bubble to look to can jump to info.)

## Configuration
[comment]: <> (TODO: Need to fill out this section.)