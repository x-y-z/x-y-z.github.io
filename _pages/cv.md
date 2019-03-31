---
layout: archive
title: ""
permalink: /cv/
author_profile: true
redirect_from:
  - /resume
---

Research Interests
=====
My research interests lie at the hardware/software interface. I am interested in building programmable, high-performance, and energy-efficient systems for scale; i.e., cloud and datacenter environments. To that end, my prior research projects have focused on designing heterogeneous systems and developing operating system abstractions and mechanisms for seamless integration of accelerators (e.g. GPUs, FPGAs) in such systems. A key component of my work has been to modify the memory management core of the Linux kernel, where my code has been vetted by Linux developers and upstreamed into the mainline kernel releases since the 4.14 series.

Education
======
* **Rutgers University** --- *2012 - 2019*
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
* *Software Engineer*, **NVIDIA Research** --- *2019 - present*

* *Research Assistant*, **Rutgers University**, New Brunswick, NJ --- *2012 - 2018*
  * Integrated 1GB transparent hugepage support into Linux for minimizing address
	translation overhead of HPC workloads.
  * Developed memory defragmentation technique for Linux to support TLB coalescing
	in modern processors (e.g. AMD Ryzen) and minimize address translation overhead
	for accelerators.
  * Upstreamed transparent hugepage migration support to Linux kernel v4.14 for
	high performance heterogeneous memory management.
  * Designed a low-overhead and efficient TLB coherence protocol (HATRIC) to
	enable high performance data migration for heterogeneous memory management
	in virtualized environment

* *Research Intern*, **NVIDIA Research**, Austin, TX --- *Jan. 2017 - May 2017*
  * Studied performance implication of CPU-GPU heterogeneous systems with
	OS-managed GPU memory.
  * Implemented memory defragmentation in Linux to increase more than 10x
	address translation coverage.

* *Research Intern*, **NVIDIA Research**, Austin, TX --- *May 2016 - Sep. 2016*
  * Integrated transparent hugepage migration into Linux for high performance
	CPU-GPU heterogeneous systems.
  * Improved page migration throughput by more than 5x for high performance
	heterogeneous memory management.
  * Kernel patches: [THP migration](https://lwn.net/Articles/723764/) & [Accelerating page migration](https://lkml.org/lkml/2016/11/22/457)

* *Intern*, **VMware**, Palo Alto, CA --- *Jun. 2015 - Aug. 2015*
  * Carried out performance analysis of [Project Bonneville](https://blogs.vmware.com/cloudnative/2015/06/22/introducing-project-bonneville/), the prototype of [vSphere Integrated Containers](https://www.vmware.com/products/vsphere/integrated-containers.html).
  * Implemented the memory snapshot function of containers in [Project Bonneville](https://blogs.vmware.com/cloudnative/2015/06/22/introducing-project-bonneville/).

Skills
======
* Programming Languages & Tools
  * Have written various C/C++ programs for 10 years
  * Use Python, R for more than 4 years
  * Use Haskell, Coq for more than 1 year
* Operating systems: Linux and FreeBSD kernel hacking

