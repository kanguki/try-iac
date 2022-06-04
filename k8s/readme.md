

Using local cluster, we don't have external ip by default.
Steps below are meant to solve this issue.

# local lb: metallb

https://kind.sigs.k8s.io/docs/user/loadbalancer/

# expose service to external ip address

```
k expose deployment redis  --type=LoadBalancer #by default bind 6379 -> 6379
k delete svc redis
k expose deployment redis --port=6378 --target-port=6379  --type=LoadBalancer #we can access 6378 from external ip
```

# double check

###### loadBalancer

```
LB_IP=$(kubectl get svc/foo-service -o=jsonpath='{.status.loadBalancer.ingress[0].ip}')
for _ in {1..10}; do
  curl ${LB_IP}:5678
done
```

###### redis

```
REDIS_IP=$(kubectl get svc/redis -o=jsonpath='{.status.loadBalancer.ingress[0].ip}')
telnet $REDIS_IP 6378
```
