# RHorizon — Kubernetes starter (kind) avec base regroupée

Cette version regroupe la **base** par composants : `frontend/`, `backend/`, puis le reste (namespace, ingress, etc.).
Overlay `kind` inclus pour tester en local avec **ingress-nginx** et une DB Postgres éphémère.

## Structure
```
k8s/
  base/
    frontend/           # tout le front (SA, ConfigMap, Deploy, Service, HPA, PDB, NP)
    backend/            # tout le back  (SA, ConfigMap, Secret exemple, Deploy, Service, HPA, PDB, NP)
    namespace.yaml
    ingress.yaml        # nginx (local)
    kustomization.yaml  # référence frontend/ et backend/
  overlays/
    kind/
      kustomization.yaml
      images.patch.yaml
      ingress-kind.patch.yaml
      np-front-kind.patch.yaml
      secret-back.kind.yaml
      postgres-kind.yaml
kind-cluster.yaml
```

## Utilisation
1) Créer le cluster kind :
```bash
kind create cluster --config kind-cluster.yaml
```
2) Installer ingress-nginx :
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx   --namespace ingress-nginx --create-namespace   --set controller.allowSnippetAnnotations=true
```
3) Builder & charger vos images locales :
```bash
docker build -t rhorizon/back:dev backend/
docker build -t rhorizon/front:dev frontend/
kind load docker-image rhorizon/back:dev --name rhz
kind load docker-image rhorizon/front:dev --name rhz
```
4) Déployer :
```bash
kubectl apply -k k8s/overlays/kind
kubectl -n rhorizon get pods,svc,ingress
```
5) Accès : http://rhorizon.localtest.me/

## Passage à EKS (plus tard)
- Pousser les images dans ECR
- Remplacer l’Ingress nginx par ALB (annotations), ajouter ACM/WAF
- IRSA sur le SA backend pour Secrets Manager/S3
- Overlays nonprod/prod
