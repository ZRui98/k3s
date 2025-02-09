# Vector log agent

### Updating the agent daemonset

```bash
kubectl apply -f vector-agent.yaml # apply change
kubectl rollout restart ds vector -n vector # roll daemonset
kubectl rollout status daemonset vector -n vector # status
```