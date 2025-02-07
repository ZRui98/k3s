# Set up VictoriaMetrics

### VMOperator

Not migrating from prometheus so all prometheus converters were disabled.
kubectl apply -f monitoring.yaml

### VMSingle


### Auth for using vmui
htpasswd -c auth.txt.secret adminuser
kubectl create secret generic vmui-auth-secret --from-file=auth.txt.secret -n vm