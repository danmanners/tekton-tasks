# Buildah Build

This task builds a container image and then uses `skopeo` to copy it into a tarball to store on a shared workspace volume. If you want to use this task to build multiple platform architectures natively and simultaneously, make sure that you set the accessMode to `ReadWriteMany`.

> This task should be used in conjunction with the [buildah-push](../buildah-push/) task

## How to Use

In the context of a Tekton `Pipeline`, you would use `Task`s/`ClusterTask`'s as such:

```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: excalidraw
  namespace: tekton-pipelines
spec:
  # Paramaters
  params:
    - name: gitUrl
      type: string
      default: https://github.com/excalidraw/excalidraw.git
    - name: gitRevision
      type: string
      default: v0.12.0
    - name: containerRegistry
      type: string
      default: ghcr.io
    - name: manifestName
      type: string
      default: excalidraw
    - name: containerName
      type: string
      default: danmanners/excalidraw

  # Workspaces
  workspaces:
    - name: ws
    - name: containers

  # Tasks
  tasks:
    # Clone the source Git Repo
    - name: clone
      taskRef:
        kind: ClusterTask
        name: git-clone
      params:
        - name: url
          value: $(params.gitUrl)
        - name: revision
          value: $(params.gitRevision)
      workspaces:
        - name: output
          workspace: ws

    # Build the 'amd64' container image
    - name: amd64
      taskRef:
        kind: ClusterTask
        name: buildah-build
      workspaces:
        - name: source
          workspace: ws
        - name: containers
          workspace: containers
      runAfter:
        - clone
      params:
        - name: IMAGE
          value: "$(params.containerRegistry)/$(params.containerName):$(params.gitRevision)"
        - name: ARCH
          value: amd64
        - name: BUILD_EXTRA_ARGS
          value: "--no-cache"

    # Build the 'arm64' container image
    - name: arm64
      taskRef:
        kind: ClusterTask
        name: buildah-build
      workspaces:
        - name: source
          workspace: ws
        - name: containers
          workspace: containers
      runAfter:
        - clone
      params:
        - name: IMAGE
          value: "$(params.containerRegistry)/$(params.containerName):$(params.gitRevision)"
        - name: ARCH
          value: arm64
        - name: BUILD_EXTRA_ARGS
          value: "--no-cache"

    # Push up the container images to the image destination
    - name: push
      taskRef:
        kind: ClusterTask
        name: buildah-testing
      workspaces:
        - name: containers
          workspace: containers
      runAfter:
        - amd64
        - arm64
      params:
        - name: IMAGE
          value: "$(params.containerRegistry)/$(params.containerName):$(params.gitRevision)"
```

In order to ensure that the multi-architecture containers build on the appropriate hardware, you'll need to create the appropriate specifications on a `PipelineRun` resource.

```yaml
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: excalidraw
  generateName: excalidraw-
  namespace: tekton-pipelines
spec:
  serviceAccountName: tekton-user
  pipelineRef:
    name: clone-and-build
  taskRunSpecs:
    # Build the amd64 image on dedicated amd64 hardware
    - pipelineTaskName: amd64
      taskPodTemplate:
        nodeSelector:
          kubernetes.io/arch: amd64
    # Build the arm64 image on dedicated arm64 hardware
    - pipelineTaskName: arm64
      taskPodTemplate:
        nodeSelector:
          kubernetes.io/arch: arm64
  workspaces:
    # Disk Volumes
    - name: ws
      volumeClaimTemplate:
        spec:
          storageClassName: ceph-filesystem
          accessModes:
            - ReadWriteMany
          resources:
            requests:
              storage: 1Gi
    - name: containers
      volumeClaimTemplate:
        spec:
          storageClassName: ceph-filesystem
          accessModes:
            - ReadWriteMany # access mode may affect how you can use this volume in parallel tasks
          resources:
            requests:
              storage: 5Gi
```
