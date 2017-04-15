---
title: About
process:
    twig: true
    markdown: false
content:
    items: '@self.modular'
    order:
        by: date
        dir: desc
---

{% include 'partials/aboutme.html.twig' %}