# V1 Project Status

**Release:** Version 1 — Local CI/CD with Jenkins, Docker and Kind  
**Evidence date:** 9 November 2026  
**Jenkins run:** Build #21

## Completed and evidenced

- [x] Jenkins Build #21 completed successfully.
- [x] All seven displayed pipeline stages passed.
- [x] Jenkins displayed an overall duration of 48 seconds.
- [x] Dependency installation and test stage passed.
- [x] Docker Build stage passed.
- [x] Kubernetes Deploy stage passed.
- [x] Application Health Check stage passed.
- [x] `http://localhost:8081/health` returned `{"status":"healthy"}` in the browser capture.
- [x] Kind cluster `devops-lab` was present.
- [x] Kind node was `Ready` in the captured check.
- [x] Two application Pods were `1/1 Running` in the later captured check.
- [x] NodePort Service `devops-service` was present.

## Pending / not claimed as complete

- [ ] Docker Hub authentication and image push.
- [ ] Verify the published image tag/digest after registry push.
- [ ] Investigate the single Pod restart shown in the verification screenshot.
- [ ] Add the exact source files (application, Dockerfile, Jenkinsfile and Kubernetes manifests) to this documentation package/repository if they are not already present.

## Status statement

V1's **local pipeline and deployment path passed in the captured run**. Registry publishing remains pending. One Pod restart was observed, but its cause is not known from the evidence currently available.
