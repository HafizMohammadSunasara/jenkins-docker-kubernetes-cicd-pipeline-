# Deep Dive — End-to-End CI/CD Pipeline (V1)

> A practical engineering walkthrough of the local Jenkins → Docker → Kind Kubernetes delivery path.

## 1. Objective

The goal of V1 was to automate the path from a source-control change to a running application in a local Kubernetes cluster. Instead of manually repeating every step, Jenkins coordinates the stages and reports whether the workflow succeeds.

The project is intentionally scoped as a **local development and learning environment**. It demonstrates automation concepts, but it is not presented as a highly available or production-secure platform.

## 2. CI and CD in this project

**Continuous Integration (CI)** is represented by source checkout, dependency installation and automated tests. The pipeline should fail early when the application cannot be tested successfully.

**Continuous Delivery / Deployment (CD)** is represented by building the container image, applying the Kubernetes deployment and checking the application after rollout.

In this run, Jenkins reported that the pipeline was started by an SCM change. Build #21 completed all seven displayed stages successfully.

## 3. Runtime architecture

The working environment consisted of:

- A Windows workstation.
- Jenkins running locally in a browser at `localhost:8080`.
- A Windows-based Jenkins agent executing the pipeline workspace under `C:\JenkinsAgent\workspace\ci-cd-pipeline`.
- Docker Desktop providing the container runtime.
- A Kind cluster named `devops-lab`.
- A Kubernetes application exposed through a NodePort Service.

The evidence shows the Kind control-plane container publishing host port `8081` to container port `30080`. The service output shows `5000:30080/TCP`. Together, these mappings made the health endpoint reachable at `http://localhost:8081/health`.

### Request path

1. A client requests `http://localhost:8081/health` on the workstation.
2. Docker Desktop's published port forwards host port `8081` to port `30080` on the Kind node container.
3. The Kubernetes NodePort Service receives traffic on NodePort `30080`.
4. The Service forwards traffic to the application Pods on the configured target port.
5. The application returns the JSON body `{"status":"healthy"}`.

The exact targetPort and selector values should be confirmed from the actual Service manifest; the captured `kubectl get services` output does not display those fields.

## 4. Pipeline walkthrough

### Stage 1 — Checkout SCM

Jenkins identifies the source-control change and starts the pipeline. The stage view recorded this step as successful.

**Why it matters:** repeatable delivery starts with an identifiable source revision. For stronger traceability, record the commit SHA with every image build and deployment.

### Stage 2 — Checkout

The pipeline checks out the repository into its Jenkins workspace. The captured agent workspace was `C:\JenkinsAgent\workspace\ci-cd-pipeline`.

**Why it matters:** the build should run against a known repository revision rather than files left over from a developer's local working directory.

### Stage 3 — Install Dependencies and Test

The pipeline installed application dependencies and ran the test stage. Jenkins recorded this stage as successful and displayed a duration of 14 seconds.

**Why it matters:** automated tests provide a quality gate before the container build and deployment steps. A green stage confirms the configured test command exited successfully; it does not prove every possible behavior is covered.

### Stage 4 — Docker Build

Jenkins built the application's container image. The stage passed in 3 seconds.

**Important boundary:** Docker Hub push was removed from this run because authentication was failing. Therefore, the evidence proves that the Docker Build stage passed; it does not prove that an image was uploaded to Docker Hub.

A future implementation should use a Jenkins credential binding for the registry access token. Credentials must never be committed to Git, embedded in the Jenkinsfile, or printed to the build log.

### Stage 5 — Kubernetes Deploy

The pipeline deployed the application to the Kind cluster. Jenkins recorded the stage as successful in 18 seconds.

The later verification screenshot showed:

- Kind cluster: `devops-lab`
- Node: `devops-lab-control-plane`
- Node status: `Ready`
- Node version displayed by `kubectl`: `v1.37.0`
- Service: `devops-service`
- Service type: `NodePort`
- Service ports displayed: `5000:30080/TCP`

### Stage 6 — Application Health Check

The pipeline's health-check stage passed. A separate browser capture shows the endpoint `http://localhost:8081/health` returning:

```json
{"status":"healthy"}
```

This proves that the endpoint returned the shown body at the time of capture. It does not by itself prove sustained availability, load capacity, or full functional correctness.

### Stage 7 — Post Actions

The post-actions stage passed, and Jenkins printed:

```text
CI/CD pipeline completed successfully!
```

The full run was displayed as 48 seconds.

## 5. Kubernetes objects and their roles

### Deployment

A Deployment manages the desired number of application replicas and replaces Pods as needed. The captured later Pod listing showed two application Pods, each `1/1 Running`.

### Pods

Pods are the execution units that host the application containers. The `READY` column indicates whether the containers are ready according to Kubernetes readiness conditions. The captured later output showed both Pods ready.

One of the two Pods showed a restart count of `1`. The screenshot does not reveal why it restarted. It should be investigated rather than assigned an assumed cause.

### Service — NodePort

The Service provides a stable network endpoint for the application Pods. A NodePort exposes a port on the Kubernetes node. In this local Kind setup, the Docker port mapping connected workstation port `8081` to node port `30080`.

## 6. What the evidence proves — and what it does not

| Evidence | What it supports | What it does not establish |
|---|---|---|
| Jenkins Build #21 stage view | All seven displayed stages passed; 48-second run | That future runs will always pass |
| Docker Build stage | The configured image-build command succeeded | Docker Hub push or registry availability |
| Kubernetes Deploy stage | The deployment stage exited successfully | Production-grade availability or security |
| `kubectl get nodes` | The Kind node was `Ready` at capture time | Multi-node resilience |
| `kubectl get pods` | Two Pods were `1/1 Running` in the later output | Zero restarts; one Pod had restart count 1 |
| `kubectl get services` | NodePort Service and displayed port mapping | Exact selectors and targetPort values |
| Browser `/health` response | Endpoint returned `{"status":"healthy"}` | Full application test coverage or load performance |

## 7. Investigation: the observed Pod restart

The verification screenshot shows one Pod with a restart count of `1` and a note that the restart occurred about 90 seconds earlier. Both Pods were nevertheless `1/1 Running` in that later listing.

The root cause is **unknown from the available evidence**. To investigate it, use the current Pod name from `kubectl get pods`:

```powershell
# List current Pods and restart counts
kubectl get pods -o wide

# Inspect events, current state and previous termination details
kubectl describe pod <current-pod-name>

# Read logs from the previous container instance, if available
kubectl logs <current-pod-name> --previous

# Read current container logs
kubectl logs <current-pod-name>

# Inspect recent namespace events
kubectl get events --sort-by=.metadata.creationTimestamp
```

Do not copy the illustrative `<current-pod-name>` literally; substitute the exact name returned by `kubectl get pods`.

## 8. Registry publishing — planned improvement

A registry push step is intentionally absent from the successful V1 run. To complete it safely:

1. Generate a Docker Hub access token with the minimum required permissions.
2. Add the Docker Hub username and token as a Jenkins credential.
3. Reference the credential by ID in the Jenkinsfile's credential-binding mechanism.
4. Use a unique image tag (build number and/or commit SHA).
5. Authenticate without placing the token in command text or logs.
6. Push the image and verify the repository tag or digest in Docker Hub.
7. Update the deployment strategy to pull the published image, or document why the local Kind image-load workflow is retained.

Avoid using `latest` as the only deployment identifier. Immutable tags make rollbacks and investigations easier.

## 9. Engineering improvements for later versions

These are roadmap items, not features claimed as complete in V1:

- Pin application dependencies and use a reproducible dependency lock strategy.
- Add explicit image tagging and registry publishing.
- Add test reports and build artifacts to Jenkins.
- Add Kubernetes rollout status checks and a documented rollback procedure.
- Add resource requests/limits and meaningful readiness/liveness probes after reviewing the actual manifests.
- Move from local Kind deployment to AWS EKS in V2.
- Add Helm and GitOps with Argo CD in V3.
- Add image and source security scanning plus monitoring in V4.

## 10. Final assessment

V1 achieved its core local automation objective: the captured Jenkins build passed all seven stages, the Docker build stage succeeded, the deployment stage succeeded, the Kind node was Ready, two Pods were Ready/Running in the later check, and the application health endpoint returned the expected JSON body.

The registry push remains an explicit gap. The Pod restart is an observed event with an undetermined cause. Recording both facts makes this project report more useful and credible than hiding them.
