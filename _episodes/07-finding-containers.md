---
title: "Finding container images"
teaching: 10
exercises: 5
questions:
- "How do I find existing container images?"
- "How can I make use of them?"
objectives:
- "Be aware of good locations for container images"
keypoints:
- "Container images are available from different registries."
- "The images can be found in different ways."
- "There are most likely already images available that work for your analysis."
- "You can build on top of those images."
---
When wanting to create a container image for your analysis, starting from
scratch can be quite challenging since it can involve installing the correct
system packages, creating new users, defining entrypoints and a lot more.
However, this is also typically not needed.
Instead, it is very likely that there are already images available that cover
your needs.
In the following, you will get an idea where you can find suitable images.

## Common pitfalls

Before pointing you to locations where and how you can find images and their
corresponding `Dockerfiles`, a few words of warning.
You will find several tutorials on building container images online that
target web developers and data scientists who are using generic Javascript
and Python libraries.
For example, for Python development, they will recommend to use a
[Python image on Docker Hub][dockerhub-python]
(more on Docker Hub below).
While this makes it easy to obtain a Python version in a software container,
this is _not_ suitable for the majority of applications in HEP.

One of the main reasons is that such images are based on Linux distributions
that are (very) different from those used in HEP environments.
For example, to be able to read ROOT files on the Grid or EOS,
you might rely on XRootD and a Kerberos token or VOMS proxy.
Installing and configuring the required packages on Debian or Alpine Linux
(the typical distributions used for the Python images), can get very
complicated or might even be impossible.
Furthermore, since the focus in the example is on the Python version, but not
the underlying Linux distribution, it can happen that this distribution changes
from one day to the other.
To avoid such issues, follow the advice listed below.

## CAT-supported frameworks

The CMS Common Analysis Tools (CAT) group supports several
[analysis frameworks][cat-frameworks]
with different target groups.
The majority of those frameworks will have a container image that you can
directly use for your analysis.
How to do that is part of the
[GitLab CI for CMS lesson][gitlab-cms-lesson].
These images should also be available on CVMFS on `unpacked.cern.ch`.
Should those images not be listed in the documentation, you can find them
through the GitLab project:

1. On the GitLab project/repository page, click "Deploy" on the left-hand side menu
2. Select "Container registry"

On the page that opens, you will see all the available container images
and their respective tags.

Other frameworks often also have container images available.
You can find them in a similar way.
A good indication of the availability of a container image for an analysis
framework is the existence of a `Dockerfile` in the repository.
There is unfortunately no container image search available for the GitLab
container image registry.
However, you can find a large number of images used at CERN in the
[CERN unpacked sync repository][unpacked-sync-recipe].

## CMS base images

CMS Offline and Computing provides base images to be used with CMSSW on the grid.
The source for these images is in the
[CMSSW GitHub area][cms-oc-docker],
which also links to their Docker Hub location.
They are used for the
[CMSSW Singularity][cms-singularity] setup.

Another alternative, which typically also has more utility libraries and tools
installed already, are the
[CMS Open Data container images][cms-containers].
They often provide a good starting point to build your own images.

## CERN base images

If you need to build your own image from scratch, the next-best recommendation
is to use the [(Alma) Linux images][cern-linux-images]
provided by CERN IT.
In contrast to bare Alma Linux images, they already have some of the CERN
software repositories configured such that installing CERN software should be
easier.
Mind, however, that these images contain very little packages and you will have
to install them yourself.

## Docker Hub

[Docker Hub][docker-hub] is the original container image registry.
You will find a huge number of images for all kinds of use cases on it.
Make sure to check the build dates of the available images to avoid using very
old ones.
Furthermore, make sure to check the build logs that are linked in most cases
to understand which software is installed into those images.
There are often links provided to the Git repositories from which these images
have been built.

> ## Using software images from the internet is dangerous
>
> It is highly recommended to only use the _official_ images from Docker Hub.
> While running a container image on your system is in principle safe since
> it is isolated from the rest of your computer, this can change as soon as
> you enter/mount your personal credentials into a running container.
>
{: .callout}

## Other registries

Besides Docker Hub, there are also other container image registries available.
An example is the GitHub Container Registry (images starting with `ghcr.io`),
where the image names point to the GitHub repositories from which the container
images have been built.
In those repositories, you will again most likely find the corresponding
`Dockerfile`.

{% include links.md %}
