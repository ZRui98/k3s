# Set up VictoriaMetrics

### VMOperator

Not migrating from prometheus so all prometheus converters were disabled.
```bash
kubectl apply -f monitoring.yaml
```

### VMSingle


### Auth for using vmui

```bash
htpasswd -c auth.txt.secret adminuser
kubectl create secret generic vmui-auth-secret --from-file=auth.txt.secret -n vm
```