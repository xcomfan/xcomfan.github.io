---
layout: page
title: "Git Reference"
permalink: /gitref/
---

# Workflow

[comment]: <> (TODO: Once you build out the content may want to sue the subgraph option to make more granular pages.)

{% mermaid %}
 flowchart LR

    init([Create or clone repo])-->work([Work on code])
    init-->history([View Repository History])
    history-->work
    work-->commit([Commit your changes])

    click init "{% link gitref/init.md %}"
    click work "{% link gitref/work.md %}"
    click history "{% link gitref/history.md %}"
    click commit "{% link gitref/commit.md %}"
{% endmermaid %}

[comment]: <> (TODO: May want to add some quick links so that someone who does not know whic mermaid bubble to look to can jump to info.)

# Configuration
[comment]: <> (TODO: Need to fill out this section.)