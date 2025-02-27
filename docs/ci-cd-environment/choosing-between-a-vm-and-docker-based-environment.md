---
Description: Semaphore supports two types of environments for running jobs. A Virtual Machine-based environment and a Docker container-based environment.
---

# Choosing Between VM and Docker-based Environments

Semaphore supports two types of environments for running jobs. A Virtual Machine-based 
environment and a Docker container-based environment.

- [Virtual Machine-based environments](#virtual-machine-based-environments)
- [Docker-based environments](#docker-based-environments)

Both environment types have their respective advantages and disadvantages.
Use the following guides to choose the best environment type for your project.

- [When should you use Virtual Machines for your projects?](#when-should-you-use-virtual-machines-for-your-projects)
- [When should you use Docker-based environments?](#when-should-you-use-docker-based-environments)

Sometimes the choice between Virtual Machines and Docker containers is not clear.
Commonly asked questions include:

- [Can I use Docker images in Virtual Machines?](#can-i-use-docker-images-in-virtual-machines)
- [Can I build Docker-in-Docker?](#can-i-build-docker-in-docker)

## Virtual Machine-based environments

Semaphore's virtual machines are maintained by our engineers. Every two weeks, a
new release of the platform is released, bundled with the latest software
packages.

To use a virtual machine-based environment for your jobs, use the following type
of agent definition in your pipeline YAML file.

**Linux-based virtual machines**

``` yaml
agent:
  machine:
    type: e1-standard-2
    os_image: ubuntu1804
```

**Mac-based virtual machines**

``` yaml
agent:
  machine:
    type: a1-standard-4
    os_image: macos-xcode12
```

We also have helpful documentation about [Machine Types][machine-types] and the
[Ubuntu 18.04][ubuntu1804], [Mac OS XCode 11][xcode11], and [Mac OS XCode 12][xcode12] virtual machines images.
Have a look!

## Docker-based environments

A Docker-based environment is a composable environment that allows the use
of one or more Docker containers to construct your test environment on
Semaphore.

The recommended way of using Docker images is to build and maintain a
custom-built image with only the software that is necessary for your
project.

Alternatively, if you are just starting out with Docker-based environments, you
can use one of Semaphore's convenience images.

To use a container-based environment, define at least one container in your
agent definition in your pipeline YAML file.

``` yaml
agent:
  machine:
    type: e1-standard-2

  containers:
    - name: main
      image: 'registry.semaphoreci.com/ruby:2.6'

    - name: db
      image: 'registry.semaphoreci.com/postgres:9.6'
      env_vars:
        - name: POSTGRES_PASSWORD
          value: keyboard-cat

    - name: cache
      image: 'registry.semaphoreci.com/redis:5.0'
```

Read more about [docker-based environments][docker-based].

!!! info "Semaphore convenience images redirection"
	Due to the introduction of [Docker Hub rate limits](/ci-cd-environment/docker-authentication/), if you are using a [Docker-based CI/CD environment](/ci-cd-environment/custom-ci-cd-environment-with-docker/) in combination with convenience images, Semaphore will **automatically redirect** any pulls from the `semaphoreci` Docker Hub repository to the [Semaphore Container Registry](/ci-cd-environment/semaphore-registry-images/).	

## When should you use Virtual Machines for your projects?

- **You want to have an up-to-date testing environment with the latest software
  packages.** The Virtual Machines are regularly updated with the latest software 
  packages by engineers at Semaphore.

- **You are building Docker images or spinning up local Kubernetes testing
  clusters in CI/CD jobs.** When working with Docker or Kubernetes directly,
  you usually need full control over sockets, network-devices, and subnets.
  Access to these is provided in VMs, but not in containers.

- **You are running or testing nested Virtual Machines**. In these situations,
  you need full control over the operating system, which only virtual machines
  provide.

## When should you use Docker-based environments?

- **You want to have full control over the software installed in your test
  environment**. Rolling software updates offered in the VMs are a great way to
  keep your test environment up-to-date, but they can also introduce unwanted
  issues into your test suite when a new release is rolled out. If you build and
  maintain your own Docker images, you gain full control over the release cycle
  of your testing environment.

- **You want to have the same testing environment on Semaphore as in your local
  development environment**. A docker image is a perfect choice in this case.
  With Docker images, you can guarantee the same software stack across
  development and build environments.

- **You need custom software that is not available in the VM images**.
  Semaphore's Virtual Machines are packed with a wide variety of languages and
  tools, but they don't have everything. In the virtual machines, you
  can use `apt-get` to install what you need, but a more efficient way is to build a
  docker image that pre-installs everything that is necessary for your tests.

## Can I use Docker images in Virtual Machines?

Yes. Running Docker images in Virtual Machines is one of the most common ways to
start background services necessary for your jobs.

Instead of defining them in the container section in the agent, use the
prologue commands to spin up your Docker images.

For example, to start RabbitMQ in a Virtual Machine, use the following snippet:

``` yaml
blocks:
  - name: Tests
    task:
      prologue:
        commands:
          - docker run -d --hostname my-rabbit --name rabbit -p 15672:15672 -p 5672:5672 rabbitmq:3-management
          - sleep 5 # give some time for Rabbit to start up

      jobs:
        - name: Rabbit Test
          commands:
            - make rabbit.based.tests
```

## Can I build Docker-in-Docker?

It is possible to build Docker images in container-based environments 
(Docker socket is mounted and available), but it is not the 
recommended way to build Docker images on Semaphore.

Docker-in-Docker has a set of issues that you need to be aware of before
choosing this approach.

Jérôme Petazzoni — the author of the feature that made it possible for Docker to
run inside a Docker container — wrote a [blog post about why you shouldn't do it][blog-docker-in-docker].
The use case he describes matches the use case of a CI Docker container that
needs to run jobs inside other Docker containers.

A lot has changed since 2015 in the Docker world. However, some issues are still
present.

We recommend using Virtual Machines for building Docker images.

[machine-types]: https://docs.semaphoreci.com/ci-cd-environment/machine-types/
[ubuntu1804]: https://docs.semaphoreci.com/ci-cd-environment/ubuntu-18.04-image/
[xcode11]: https://docs.semaphoreci.com/ci-cd-environment/macos-xcode-11-image/
[xcode12]: https://docs.semaphoreci.com/ci-cd-environment/macos-xcode-12-image/
[docker-based]: https://docs.semaphoreci.com/ci-cd-environment/custom-ci-cd-environment-with-docker/
[blog-docker-in-docker]: https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/
