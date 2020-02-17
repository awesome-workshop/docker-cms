---
title: "Using Singularity"
teaching: 10
exercises: 0
questions:
- "How can I use CMSSW inside a container on LXPLUS?"
objectives:
- "First learning objective. (FIXME)"
keypoints:
- "First key point. Brief Answer to questions. (FIXME)"
---

The previous episode has given you an idea how complicated it can be
to run containers with CVMFS access on your computer. However, at the
same time it gives you the possibility to develop code on a computer
that doesn't need to know anything about CMS software in the first place.
The only requirement is that Docker is installed.

You will also have noticed that in several cases *privileged* containers
are needed. These are not available to you on LXPLUS (nor is the `docker`
command). On LXPLUS, the tool to run containers is Singulariy.

- point to CMS documentation
- point out what mounted/available within Singularity
- run the build script in Singularity

{% include links.md %}

