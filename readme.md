# Dan Manners' Tekton Tasks

The structure of this project is loosely based on the [official Tekton CI/CD Catalog repository](https://github.com/tektoncd/catalog).

## Project Structure

1. Each resource follows the following structure:

```
/task                                       👈 the kind of the resource

    /buildah-build                          👈 definition file must have same name

        /readme.md                          👈 owners of this tasks and changelog

        /0.1
            /readme.md                      👈 instructions on how to use the task

            /buildah-build.yaml             👈 should match task name; installs Task (single namespace)

            /kustomization.yaml             👈 install the resource as a ClusterTask (all namespaces)

        /0.2/
            ...
```

1. Resource YAML file includes following changes
  *  Labels include the version of the resource.
  *  Annotations include `minimum pipeline version` supported by the resource,
     `tags` associated with the resource and `displayName` of the resource

```yaml
labels:
    tekton.dev/version: "0.1"                         👈 Version of the resource

annotations:
    tekton.dev/pipelines.minVersion: "0.12.1"         👈 minVersion of pipeline resource is compatible
    tekton.dev/categories: CLI                        👈 comma separated list of categories
    tekton.dev/displayName: "Ansible Tower Cli"       👈 displayName can be optional
    tekton.dev/platforms: "linux/amd64,linux/s390x"   👈 comma separated list of platforms, can be optional

spec:
description: |-
    ansible-tower-cli task simplifies
    workflow, jobs, manage users...                   👈 Summary

    Ansible Tower (formerly ‘AWX’) is a ...
```

## List of Categories

The list of categories I am using are based on [the official Tekton CI/CD catalog list, which can be found here](https://github.com/tektoncd/hub/blob/b36255f9a512b9f4cccb4fd566a45da4d878810a/config.yaml#L15). While similar, they do not match at this time.

```yaml
categories:
  - name: Automation
  - name: Build Tools
  - name: CLI
  - name: Code Quality
  - name: Containerization
  - name: Continuous Integration
  - name: Deployment
  - name: Developer Tools
  - name: Integration & Delivery
  - name: Git
  - name: Kubernetes
  - name: Messaging
  - name: Monitoring
  - name: Networking
  - name: Publishing
  - name: Security
  - name: Testing
```
