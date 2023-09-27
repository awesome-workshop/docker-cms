---
title: "Introduction"
teaching: 10
exercises: 0
questions:
- "Which options are available to run CMSSW in a container?"
objectives:
- "Be aware of what container images exists and their individual caveats."
keypoints:
- "Full CMSSW release containers are very big."
- "It is more practical to use light-weight containers and obtain CMSSW via CVMFS."
- "The centrally supported way to run CMSSW in a container is using Singularity."
---
CMS does not have a clear concept of separating analysis software from the rest
of the experimental software stack such as event generation, data taking, and
reconstruction. This means that there is just one *CMSSW*, and the releases
have a size of several Gigabytes (>20 GB for the last releases).

From the user's and computing point of view, this makes it very impractical to
build and use images that contain a full CMSSW release. Imagine running
several hundred batch jobs where each batch node first needs to download
several Gigabytes of data before the job can start, amounting to a total of
tens of Terabytes. These images, however, can be useful for offline code
development, in case CVMFS is not available, as well as for overall
preservation of the software.

An alternative to full CMSSW release containers are Linux containers that
only contain the underlying base operating system (e.g. Scientific Linux 5/6
or CentOS 7/8) including additionally required system packages. The CMSSW
release is then mounted from the host system on which the container is
running (which could be your laptop, a GitLab runner, or a Kubernetes node).
These images have a size of a few hundred Megabytes, but rely on a good
network connection to access the CVMFS share.

One thing that has not been covered in detail in the
[introduction to Docker][intro-docker-lesson] is that containers do not
necessarily have to be executed using Docker. There are several so-called
container run-times that allow the execution of containers. CMS uses
[Apptainer (previously Singularity)][apptainer-docs] for sample production, and
[use of Singularity is also centrally supported and documented][cms-singularity].
The main reason for that is that Singularity is popular in high-performance
and high-throughput computing and does not require any root privileges.

While executing images on LXPLUS and HTCondor is more practical with
Apptainer/Singularity, running in GitLab CI is by default done using Docker.
Since Apptainer uses a proprietary image format, but supports reading and
executing Docker images, building images is better done using Docker.

{% include links.md %}
