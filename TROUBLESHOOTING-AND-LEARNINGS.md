# Troubleshooting and Engineering Learnings

## Docker Hub authentication — pending

During V1 implementation, the Docker push step produced an authentication error. The step was removed from the successful pipeline run so the local image-build and Kubernetes deployment workflow could be completed and verified.

This is a scope limitation, not a successful registry integration. The next attempt should use a Docker Hub access token stored in Jenkins Credentials, with a credential ID referenced by the pipeline. Never commit the token or print it in logs.

## Pod restart — investigation required

The later Kubernetes verification output showed two Pods as `1/1 Running`; one Pod had a restart count of `1`. The evidence does not show why this happened.

Recommended checks:

```powershell
kubectl describe pod <current-pod-name>
kubectl logs <current-pod-name> --previous
kubectl get events --sort-by=.metadata.creationTimestamp
```

Possible causes should be determined from the actual termination state, previous container logs and events. Do not assume an out-of-memory kill, failed probe or application exception without evidence.

## Operational lessons

- A green Jenkins pipeline is useful evidence, but it should be paired with post-deployment verification.
- A `Running` Pod with `1/1` readiness is a point-in-time state, not a long-term availability guarantee.
- Restart counts are valuable diagnostic signals and should be recorded rather than hidden.
- Building an image and pushing an image to a registry are separate operations; passing the first does not prove the second.
- A registry credential belongs in Jenkins Credentials, not source control.
- Keep build and deployment tags traceable to a build number or commit SHA.
