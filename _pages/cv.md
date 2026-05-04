---
layout: archive
title: "James Marsh — CV"
permalink: /cv/
author_profile: true
redirect_from:
  - /resume
---

{% include base_path %}

PI's CV. Linked from the [People page](/people/james-marsh). A PDF version is available [here](/files/cv.pdf). *(Drop your CV PDF into the `files/` directory at that path.)*

Education
======
* Ph.D., *Field*, *Institution*, *Year*
* M.Sc., *Field*, *Institution*, *Year*
* B.Sc., *Field*, *Institution*, *Year*

Positions
======
* *Year–present* — *Role*, *Institution*
* *Year–Year* — *Previous role*, *Previous institution*

Research interests
======
* Archaeal anti-phage defense systems
* Anti-defense and counter-defense strategies in archaeal viruses
* Comparative genomics of microbial conflict
* Microbiome ecology and the molecular basis of community composition

Awards & funding
======
* *Year* — *Award or grant name*

Publications
======
  <ul>{% for post in site.publications reversed %}
    {% include archive-single-cv.html %}
  {% endfor %}</ul>

Talks
======
  <ul>{% for post in site.talks reversed %}
    {% include archive-single-talk-cv.html  %}
  {% endfor %}</ul>

Teaching
======
  <ul>{% for post in site.teaching reversed %}
    {% include archive-single-cv.html %}
  {% endfor %}</ul>

Service
======
* *Reviewer* — list of journals
* *Committee* — list of committees or working groups
* *Mentorship* — students and postdocs supervised
