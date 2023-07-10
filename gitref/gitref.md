---
layout: page
title: "Git Reference"
permalink: /gitref/
---

[comment]: <> (TODO: Fix metadata around the site and see if you can update the styling to have a breadcrumb at the top.)
[comment]: <> (TODO: Good idea to add table of contents at top of each page so you can jump to sections.)

## Workflow

[comment]: <> (TODO: Once you build out the content may want to sue the subgraph option to make more granular pages.)
[comment]: <> (TODO: Now that I am working my way through I think this chart needs a complete overhaul and most of the content is in a working locally sub-graph)

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
    click remotes "{% link gitref/remotes.md%}"

    init-->work_local_sub
    work_local_sub-->remotes
    
{% endmermaid %}

[comment]: <> (TODO: May want to add some quick links so that someone who does not know which mermaid bubble to look to can jump to info.)
