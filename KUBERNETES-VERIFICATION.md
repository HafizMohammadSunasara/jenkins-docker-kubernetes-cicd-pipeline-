# Kubernetes Verification Notes

## Captured environment

| Check | Captured value |
|---|---|
| Kind cluster | `devops-lab` |
| Node | `devops-lab-control-plane` |
| Node status | `Ready` |
| Node version shown | `v1.37.0` |
| Deployment Pods in later output | 2 |
| Pod readiness/status | Both `1/1 Running` |
| Pod restarts | One Pod showed `1`; the other showed `0` |
| Service | `devops-service` |
| Service type | `NodePort` |
| Service ports shown | `5000:30080/TCP` |
| Host access | `localhost:8081` |
| Health path | `/health` |

## Commands used for verification

```powershell
kind get clusters
kubectl cluster-info --context kind-devops-lab
docker ps --format "table {{.Names}}\t{{.Ports}}"
kubectl get nodes
kubectl get pods
kubectl get services
```

## Health check

```powershell
Invoke-RestMethod http://localhost:8081/health
```

Captured response body:

```json
{"status":"healthy"}
```

## Follow-up investigation for the restarted Pod

```powershell
kubectl get pods
kubectl describe pod <current-pod-name>
kubectl logs <current-pod-name> --previous
kubectl get events --sort-by=.metadata.creationTimestamp
```

Replace the placeholder with the current Pod name. The available screenshot does not identify the restart reason, so the root cause remains open.
