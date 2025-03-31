---
title: "Containers based on CVMFS"
teaching: 15
exercises: 10
questions:
- "How can I get a more light-weight CMSSW container?"
- "What are the caveats of using a light-weight CMSSW container?"
objectives:
- "Understand how the light-weight CMS containers can be used."
keypoints:
- "The light-weight CMS containers need to mount CVMFS."
- "They will only work with CVMFS available."
---
In a similar way to [running CMSSW in GitLab][gitlab-cms-lesson], the
images containing only the base operating system (e.g. Scientific Linux 5/6
or CentOS 7/8) plus additionally required system packages can be used to run
CMSSW (and other related software). CMSSW needs to be mounted via CVMFS.
**This is the recommended way!**

For this lesson, we will continue with the repository we used for the
[GitLab CI for CMS lesson][gitlab-cms-lesson] and just add to it.

## Adding analysis code to a light-weight container

Instead of using these containers only for compiling and
[running CMSSW][gitlab-cms-lesson-running], we can add our (compiled) code to
those images, building on top of them. The advantage in doing so is that you
will effectively be able to run your code in a version-controlled sandbox, in
a similar way as grid jobs are submitted and run. Adding your code on top of
the base image will only increase their size by a few Megabytes. CVMFS will
be mounted in the build step and also whenever the container is executed.

Docker containers can be built in GitLab CI jobs using the [kaniko](https://github.com/GoogleContainerTools/kaniko) tool.
Additionally, [skopeo](https://github.com/containers/skopeo) cab be used to
tag the container images.

Luckily, templates are provided in the Common Analysis Tools (CAT) managed GitLab area
[cms-analysis](https://cms-analysis.docs.cern.ch/code/), which make creating the GitLab CI 
jobs quite easy to write.

### Creating a Dockerfile
Before using those templates, however, we need to write a Dockerfile, containing the commands
needed to build our code. Please refer to this [introduction][intro-docker-lesson] to learn about 
Docker and how to write a Dockerfile.

The *Dockerfile* required looks as follows:

~~~
FROM gitlab-registry.cern.ch/cms-cloud/cmssw-docker/cc7-cms:latest

ENV CMS_PATH /cvmfs/cms.cern.ch
ENV CMSSW_RELEASE CMSSW_10_6_8_patch1
ENV SCRAM_ARCH slc7_amd64_gcc820

COPY ZPeakAnalysis /ZPeakAnalysis

RUN shopt -s expand_aliases && \
    set +u && source ${CMS_PATH}/cmsset_default.sh && set -u  && \
    export SCRAM_ARCH=${SCRAM_ARCH} && \
    cmsrel ${CMSSW_RELEASE} && \
    cd ${CMSSW_RELEASE}/src && \
    cmsenv && \
    mkdir -p AnalysisCode && \
    cp -r /ZPeakAnalysis AnalysisCode && \
    scram b
~~~
{: .language-plaintext}

This is pretty complicated, so let's break this into smaller pieces.

The `FROM` directive defines the image we build on top of.
The `ENV` directives define environment variables that will be used during the build.
The `COPY` directive copies copies the local directory `ZPeakAnalysis` into a specified
location in the container. You have to remember that this Dockerfile will be used to 
build a container image in the context of a GitLab CI job, so the `ZPeakAnalysis` 
directory will be available, as the first thing the GitLab CI job does is clone the 
repository.
Finally, the `RUN` directive contains the commands that need to be run to setup the 
CMSSW developer area, move the `ZPeakAnalysis` code in a `[subsystem]/[package]` structure,
and compile with `scram`.

> ## If you want to be able to build the container on your laptop you need CVMFS at build time
> You can in principle build the container locally, but you need to mount CVMFS (which 
> must be available on the host machine) at image build time.
> Docker does not allow to mount volumes at build time, only at runtime, unless one 
> uses [docker-compose](https://docs.docker.com/compose/).
> A more straightforward way is to use [`podman`](https://podman.io/), instead of Docker, 
> to build the image locally, since `podman` allows mounting volumes at buildtime.
> ~~~
> cd [the directory where you cloned your repository]
> podman build . -v /cvmfs:/cvmfs --format docker -t [somename]
> ~~~
> {: .language-bash}
> The `build .` command tells `podman` to build the current directory, where it will look for a file called 
> `Dockerfile`. The `-v /cvmfs:/cvmfs` instructs `podman` to mount directory `/cvmfs` on the host to directory 
> `/cvfms` on the container (the syntax is `-v [host path]:[dest path]`). Finally, the `--format docker` option
> instructs `podman` to create a docker image, rather than [`oci`](https://opencontainers.org/).
{: .callout}

### Building containers in the GitLab CI
Docker images can be built in the CI using the `kaniko` image builder.
This tool allows building images from a Docker file, while running inside a container.
Using `kaniko` circumvents the problem we were alluding to before, that CVMFS needs to be 
available at build time, which cannot be achieved with 'vanilla' docker commands.

> Up until October 2023 one could build containers with CVMFS mounted at build time in the 
> GitLab CI via the use of dedicated, so called *docker-provileged* GitLab runners.
> These runners, however, have been decommissioned, see [here](https://cern.service-now.com/service-portal?id=outage&n=OTG0078219).
> Since then, the recommended way to build docker images needing CVMFS at build time is via `kaniko`.
{: .discussion}

CAT provides GitLab CI job templates for building images with `kaniko` and for tagging them (with `skopeo`).
The templates are hosted in the [`cms-analysis/general/container-image-ci-templates`](https://gitlab.cern.ch/cms-analysis/general/container-image-ci-templates
) project.

A `.gitlab-ci.yml` file using the templates for building the image is the following.

~~~
include:
  - project: 'cms-analysis/general/container-image-ci-templates'
    file:
      - 'kaniko-image.gitlab-ci.yml'
      - 'skopeo.gitlab-ci.yml'

build_image:
  stage: build
  extends: .build_kaniko
  variables:
    DOCKER_FILE_NAME: "Dockerfile"
    REGISTRY_IMAGE_PATH: "${CI_REGISTRY_IMAGE}:${CI_COMMIT_SHORT_SHA}"
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
      variables:
        PUSH_IMAGE: "true"
    - when: always
      variables:
        PUSH_IMAGE: "false"     
  tags:
    - cvmfs

tag_image:
  stage: tag
  extends: .tag_skopeo
  variables:
    IMAGE_ORIGIN_TAG: "${CI_REGISTRY_IMAGE}:${CI_COMMIT_SHORT_SHA}"
    IMAGE_DESTINATION_TAG: "${CI_REGISTRY_IMAGE}:latest"
~~~
{: .language-yaml}

This is pretty complicated, so let us break it down to smaller pieces.

The `include` statement imports the templates.

Those templates define jobs called `.build_kaniko` and `.tag_skopeo`, 
which are then used to define the jobs that are actually run, i.e. 
`build_image` and `tag_image` (those two jobs `extend`, i.e. include the 
ones defined in the templates).

The `build_image` job defines the variable `DOCKER_FILE_NAME`, identifying the
name of the Dockerfile, and `REGISTRY_IMAGE_PATH`, declaring the registry where
the image will be published. The `REGISTRY_IMAGE_PATH` variable itself points to a value 
that is formed using other variable that GitLab pre-defines for every CI job,
(along with many others, see [here](https://docs.gitlab.com/ci/variables/predefined_variables/)).
The `CI_REGISTRY_IMAGE` variable points to the container registry associated with the repository.
In case of the CERN GitLab installation, this usually corresponds to 
`gitlab-registry.cern.ch/[namespace]/[project]`.
Notice the rules for this job: the job will publish the image to the registry
only if the CI runs on the default branch (`master` or `main`, typically).

The `tag_image` job will take the image with the name specified in `IMAGE_ORIGIN_TAG`
and copy it to the name specified in `IMAGE_DESTINATION_TAG`.
This job only runs on the default branch (this behavior is set directly in the `.tag_skopeo`
job)

Since developing this using GitLab is tricky, the next episodes will cover
how you can develop this interactively on LXPLUS using Singularity or on
your own computer running Docker.
The caveat of using these light-weight images is that they cannot be run
autonomously, but always need to have CVMFS access to do anything useful.



{% include links.md %}
