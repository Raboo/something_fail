---
title: About
process:
    markdown: true
    twig: true
twig_first: false
content:
    items: '@self.modular'
    order:
        by: date
        dir: desc
---

{% include 'partials/aboutme.html.twig' %}