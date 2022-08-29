# Copy Files

This task copies one or more files from one source workspace to a destination workspace. The workspaces for source and destination can should be separate workspaces.

## How to Use

In the context of a Tekton `Pipeline`, you would use this `Task`/`ClusterTask` as such:

```yaml
---
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: copy-file
spec:
    # Workspaces
    workspaces:
        - name: ws
        - name: dockerfile
    # Tasks
    tasks:
        - name: copy-file
        taskRef:
            kind: ClusterTask
            name: copy-files
        workspaces:
            - name: source
            workspace: dockerfile
            - name: dest
            workspace: ws
        runAfter:
            - clone
        params:
            - name: source-files
            value: Dockerfile
            - name: dest-files
            value: Dockerfile
```
