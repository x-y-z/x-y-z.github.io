---
layout: archive
title: "CV"
permalink: /cv/
author_profile: true
redirect_from:
  - /resume
---

{% include base_path %}

Education
======
* **Rutgers University** --- *2012 - 2018 (expected)*
  * Ph.D in Computer Science
* **University of Pennsylvania** ---  *2011 - 2012*
  * Master in Computer Information & Science
* **University of Texas at San Antonio** --- *2009 - 2011*
  * Ph.D Candidate in Computer Science
* **Beijing University of Posts and Telecommunications** --- *2005 - 2009*
  * B.E. in Computer Science and Technology

Publications
======
  <ul>{% for post in site.publications %}
    {% include archive-single-cv.html %}
  {% endfor %}</ul>

Work experience
======
* *Research Assistant*, **Rutgers University**, New Brunswick, NJ --- *2012 - present*
  * Developing memory defragmentation for Linux to support TLB coalescing
	feature in modern processors (e.g. AMD Ryzen)
  * Proposed a hardware TLB coherence protocol to eliminate translation
	coherence overhead, which is a big bottleneck in heterogenous memory systems

* *Research Intern*, **NVIDIA Research**, Austin, TX --- *Jan. 2017 - May 2017*
  * Implmented memory defragmentation in latest Linux kernel to maximize
	translation contiguity for TLB coalescing.
  * TLB translation coverage is increased by more than 3x

* *Research Intern*, **NVIDIA Research**, Austin, TX --- *May 2016 - Sep. 2016*
  *   * Implmented transparent hugepage (THP) migration in latest Linux kernel
  * Improved Linux page migration throughput by up to 5x
  * Kernel patches under submission: [__*THP migration*__](https://lwn.net/Articles/723764/) & [__*Accelerating page migration*__](https://lkml.org/lkml/2016/11/22/457)

* *Intern*, **VMware**, Palo Alto, CA --- *Jun. 2015 - Aug. 2015*
  * Performance analysis of Project Bonneville, the foundational technology for
	vSphere Integrated Containers
  * Implemented the initial version of the runtime layer, which enables memory
	snapshot of containers in Project Bonneville
  
Skills
======
* Programming Languages & Tools
  * Have written various C/C++ programs for 10 years
  * Use Python, R for more than 4 years
  * Use Haskell, Coq for more than 1 year
* Operating systems: Linux and FreeBSD kernel hacking

