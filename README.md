# RHorizon — KIND overlay (frontend + backend images du registre)

Cet overlay **kind** déploie front + back en pointant sur ton registre Docker Hub :
- Front : `docker.io/neezo/rhorizon-frontend:v1`
- Back  : `docker.io/neezo/rhorizon-backend:v1` (modifie si ton repo/tag diffère)
- Ingress : `/` → front, **`/api` → back**
- Patches inclus : NET_BIND_SERVICE (front non-root), PDB/rolling, (optionnel) port front si ≠80

## Commandes

```bash
kind create cluster --config kind-cluster.yaml

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx   --namespace ingress-nginx --create-namespace   --set controller.allowSnippetAnnotations=true

kubectl apply -k k8s/overlays/kind
kubectl -n rhorizon rollout status deploy/front --timeout=180s
kubectl -n rhorizon rollout status deploy/back  --timeout=180s

curl -I http://rhorizon.localtest.me:8080/
curl -s http://rhorizon.localtest.me:8080/api/health || true
```
