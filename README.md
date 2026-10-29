# gitops-lab

Lo que ArgoCD sincroniza en el cluster GKE de `../infra` (floci-gcp). Terraform crea el cluster,
instala ArgoCD y crea la app raíz `root`, que vigila `apps/`. Todo lo de este repo llega al cluster
con `git push`.

```
apps/                     una Application por cosa a desplegar (las crea la app raíz)
  traefik.yaml            ingress controller (chart de Helm), Service LoadBalancer
  whoami.yaml             app de ejemplo
workloads/whoami/         Deployment + Service + Ingress (whoami.localhost) con Kustomize
```

## Cómo llega el tráfico

```
navegador -> localhost:80 -> lb-edge (socat, ../infra/compose.yaml)
          -> nodo k3s:80 -> ServiceLB -> Traefik -> Ingress -> Service whoami
```

ServiceLB es el balanceador de k3s: atiende los Service `type: LoadBalancer` publicando sus puertos
en el nodo, como haría un Network Load Balancer en GKE. El nodo solo es alcanzable dentro de la red
Docker `floci`, por eso `lb-edge` lo publica en la Mac.

La app raíz **no** está en este repo: la crea Terraform (`../infra/argocd.tf`). Si también estuviera
acá, Terraform y ArgoCD se la pisarían.

## Puesta en marcha

```bash
cd ../infra && terraform apply      # cluster + ArgoCD + app raíz
eval "$(./kubeconfig.sh)"
kubectl -n argocd get applications  # root y whoami
```

## Para practicar

1. Cambiá `replicas` en `workloads/whoami/deployment.yaml`, commit + push, y mirá cómo ArgoCD lo aplica
   (revisa Git cada 30s, o *Refresh* en la UI).
2. Escalá el deployment a mano con `kubectl`: `selfHeal` lo devuelve a lo que dice Git.
3. Agregá tu propia app: una carpeta en `workloads/` y su Application en `apps/`.
4. Borrá `apps/whoami.yaml` y pusheá: `prune` elimina la app y sus recursos.
5. Abrí http://whoami.localhost y mirá `kubectl -n traefik get svc traefik`: el `EXTERNAL-IP` es la
   IP del nodo que asignó ServiceLB.
6. Publicá ArgoCD por Traefik: agregá un Ingress para `argocd-server` (en `argocd.localhost`).
