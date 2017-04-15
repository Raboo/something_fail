---
title: About
process:
    markdown: true
    twig: true
content:
    items: '@self.modular'
    order:
        by: date
        dir: desc
---

{% include 'partials/aboutme.html.twig' %}