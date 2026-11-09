# Evidence Index

All images in this directory were supplied by the project author in this conversation. They have been included as evidence without adding synthetic terminal output or altering the reported results.

| File | What is visible | Evidence boundary |
|---|---|---|
| `01-jenkins-build-21-success.jpg` | Jenkins Build #21 with seven green stages, total duration shown as 48 seconds, and success message | Supports successful execution of the displayed pipeline stages in this run |
| `02-health-and-kubernetes-verification.jpg` | Browser health response; Kind cluster and node; Docker port mapping; Kubernetes Pods and Service output | Supports health endpoint response and the point-in-time Kubernetes state shown; one Pod restart is visible |

## Recommended additional evidence to capture later

- Jenkins console output showing the exact test command and test summary.
- Docker image list with the image tag used by the pipeline.
- The actual Deployment and Service YAML files.
- `kubectl describe pod` and previous-container logs for the Pod that restarted.
- Docker Hub repository page and image digest after registry publishing is successfully implemented.

Do not add edited or reconstructed terminal screenshots as if they were original runtime evidence.
