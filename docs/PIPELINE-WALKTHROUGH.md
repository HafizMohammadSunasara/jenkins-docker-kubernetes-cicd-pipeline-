# Pipeline Walkthrough

Jenkins Build #21 recorded the following stage sequence:

| Stage | Result | Recorded duration | Purpose |
|---|---|---:|---|
| Checkout SCM | Passed | 2s | React to and identify a source-control change |
| Checkout | Passed | 1s | Bring source into the Jenkins workspace |
| Install Dependencies and Test | Passed | 14s | Prepare dependencies and run automated tests |
| Docker Build | Passed | 3s | Build the application container image |
| Kubernetes Deploy | Passed | 18s | Deploy the application to the local Kind cluster |
| Application Health Check | Passed | 3s | Check application health after deployment |
| Post Actions | Passed | 0.1s | Run final pipeline actions and report success |

**Overall Jenkins duration:** 48 seconds.

## Pipeline boundaries

- The Docker image-build stage passed.
- Docker Hub push was removed before this successful run because authentication was failing.
- Kubernetes deployment and application health-check stages passed.
- The exact Jenkinsfile implementation should be kept with the repository as the source of truth. This walkthrough documents observed stage behavior rather than recreating an unseen Jenkinsfile.
