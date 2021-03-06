---
title: "Installing Athens"
date: 2018-09-20T15:38:01-07:00
weight: 2
---

The Go ecosystem has always been federated and completely open. Anyone with a GitHub or GitLab (or any other supported VCS) account can effortlessly provide a library with just a `git push` (or similar). No extra accounts to create or credentials to set up.

## A Federated Ecosystem

We feel that Athens should keep the community federated and open, and nobody should have to change their workflow when they're building apps and libraries. So, to make sure the community can stay federated and open, we've made it easy to install Athens for everyone so that:

- Anyone can run their own full-featured mirror, public or private
- Any organization can run their own private mirror, so they can manage their private code just as they would their public code

## Immutability

As you know, `go get` and most of the popular Go package managers like `dep`, `glide`, etc. will fetch packages directly from version control systems like GitHub. Even the `go get` for modules will fetch directly from version control systems! But there's a fundamental problem that can crop up and bite you.

Code in version control systems can always change even after it's been committed. For example, a package developer can run `git push -f` and overwrite a commit or tag that you depend on in your project. In that case, you'll often see checksum verification errors (for example, see [here](https://github.com/go-ole/go-ole/issues/185)). Even worse, packages can completely disappear or move out from under you, and that can break your build or even silently change your dependencies without you realizing.

_Athens prevents these issues by storing code in its own, immutable database_. Here's what happens when you run `go get`:


1. `go get` requests a module from Athens
2. Athens accepts the request and begins looking for the module
3. First, it looks in its storage. If it finds the module, Athens immediately sends it back to the `go get` client from (1)
4. If it doesn't find the module, it does this:
   1. Fetches it from the module's original location using a slightly modified `go get` command called `go mod download`
   2. Saves the module to storage
   3. Returns it to the client 
   > For example, if the requested module is `github.com/arschles/assert@v1.0.0` and Athens doesn't have that module in storage, it does a `go mod download github.com/arschles/assert@v1.0.0`, saves it to storage, and sends it back to the client

Athens never changes anything in its storage once it saves a module to storage (unless you manually change it), so the system has the following two important properties:

- _Athens will only ever call `go mod download` **once** per module version_. In other words, Athens will only hit step (4) once for any given module & version
- _Athens treats storage as write-only, so once a module is saved, it never changes, even if a developer changes it in GitHub_

## Release Scheme

We follow [semver](https://semver.org). Our Docker images are tagged to indicate stability:

* latest = the most recent stable release
* canary = the most recent build of master

We strongly recommend using a tagged release, e.g. `gomods/athens:v0.2.0`, instead of the latest or canary tags.

## Where to Go from Here

To make sure it's easy to install, we try to provide as many ways as possible to install and run Athens:

- It's written in Go, so you can easily build it yourself on almost any platform. You can also build the binary providing your own version and build time. See [here](./build-from-source)
- We provide a [Docker image](https://hub.docker.com/r/gomods/athens/) and [instructions on how to run it](./shared-team-instance)
- We provide [Kubernetes](https://kubernetes.io) [Helm Charts](https://helm.sh) with [instructions on how to run Athens on Kubernetes](./install-on-kubernetes)
