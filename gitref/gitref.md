---
layout: page
title: "Git Reference"
permalink: /gitref/
---

[comment]: <> (TODO: Good idea to add table of contents at top of each page so you can jump to sections.)

## Workflow

[comment]: <> (TODO: Once you build out the content may want to sue the subgraph option to make more granular pages.)

{% mermaid %}
 flowchart LR

    init([Create or clone repo])-->work([Work on code])
    init-->history([View repository history])
    history<-->work
    work-->commit([Commit your changes])
    commit-->remotes([Remote repositories])
    remotes-->init

    click init "{% link gitref/init.md %}"
    click work "{% link gitref/work.md %}"
    click history "{% link gitref/repo_history/history.md %}"
    click commit "{% link gitref/commit.md %}"
    click remotes "{% link gitref/remotes.md%}"
{% endmermaid %}

[comment]: <> (TODO: May want to add some quick links so that someone who does not know which mermaid bubble to look to can jump to info.)

## Configuration
[comment]: <> (TODO: Need to fill out this section.)