# Bernstein

## Due to issues with branches not allowing us to merge / make a PR, we took the decision while being at the Follow UP to repush everything in main, pls check the branches to see who did what

### Alexandre work on the database part: 
1. service(redis) branch
1. service(postrgres) branch

### Adrien work on the service part:
1. service(poll) branch
1. service(worker) branch
1. service(result) branch

### One APE tells to one person of the groups to push all the files into the main branch


## Tester en local (k3d : 1 master + 2 workers)

```bash
brew install k3d
k3d cluster create bernstein --agents 2 \
  -p "30021:30021@server:0" -p "30042:30042@server:0" \
  --k3s-arg "--disable=traefik@server:0"

kubectl apply -f cadvisor.daemonset.yaml
kubectl apply -f postgres.secret.yaml -f postgres.configmap.yaml -f postgres.volume.yaml -f postgres.deployment.yaml -f postgres.service.yaml
kubectl apply -f redis.configmap.yaml -f redis.deployment.yaml -f redis.service.yaml
kubectl apply -f poll.deployment.yaml -f worker.deployment.yaml -f result.deployment.yaml -f poll.service.yaml -f result.service.yaml -f poll.ingress.yaml -f result.ingress.yaml
kubectl apply -f traefik.rbac.yaml -f traefik.deployment.yaml -f traefik.service.yaml

echo "CREATE TABLE IF NOT EXISTS votes (id text PRIMARY KEY, vote text NOT NULL);" \
  | kubectl exec -i deploy/postgres -c postgres -- psql -U postgres -d mydatabase

echo "127.0.0.1 poll.dop.io result.dop.io" | sudo tee -a /etc/hosts
```

- Poll : http://poll.dop.io:30021
- Result : http://result.dop.io:30021
- Dashboard Traefik : http://localhost:30042
- cAdvisor : `kubectl -n kube-system port-forward ds/cadvisor 8080:8080` puis http://localhost:8080

Nettoyage : `k3d cluster delete bernstein`
