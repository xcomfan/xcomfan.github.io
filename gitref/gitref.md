---
layout: page
title: "Git Reference"
permalink: /gitref/
---

# Workflow

{% mermaid %}
 flowchart LR
    init([Create or clone repo])-->work([Work on code])-->commit([Commit your changes]);
    click init "{% link gitref/init.md %}"
    click work "{% link gitref/work.md %}"
    click commit "{% link gitref/history.md %}"
{% endmermaid %}

# Configuration
