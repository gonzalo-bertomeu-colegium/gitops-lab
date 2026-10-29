# gitops-lab

Lo que ArgoCD sincroniza en el cluster GKE de `../infra` (floci-gcp). El cluster y ArgoCD los crea
Terraform; todo lo de este repo llega al cluster con `git push`.

```
bootstrap/root-app.yaml   app raíz: se aplica a mano una vez, vigila apps/
apps/                     una Application por cosa a desplegar
workloads/whoami/         app de ejemplo con Kustomize
```

## Puesta en marcha

```bash
# 1. Remoto en GitHub (público: ArgoCD lo clona sin credenciales)
gh repo create gitops-lab --public --source . --push

# 2. kubeconfig del cluster y app raíz
eval "$(../infra/kubeconfig.sh)"
kubectl apply -f bootstrap/root-app.yaml
kubectl -n argocd get applications
```

## Para practicar

1. Cambiá `replicas` en `workloads/whoami/deployment.yaml`, commit + push, y mirá cómo ArgoCD lo aplica
   (revisa Git cada 30s, o *Refresh* en la UI).
2. Escalá el deployment a mano con `kubectl`: `selfHeal` lo devuelve a lo que dice Git.
3. Agregá tu propia app: una carpeta en `workloads/` y su Application en `apps/`.
4. Borrá `apps/whoami.yaml` y pusheá: `prune` elimina la app y sus recursos.
5. Probá la app sin ingress: `kubectl -n whoami port-forward svc/whoami 8081:80` y abrí
   http://localhost:8081.
