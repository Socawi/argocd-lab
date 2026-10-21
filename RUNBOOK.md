## Contexto
Usar este runbook para instalar Argo CD y desplegar las aplicaciónes demo en un clúster Kubernetes.

## Requisitos previos
- Acceso a un clúster Kubernetes activo y permisos para crear namespaces, recursos y CRD.
- kubectl configurado con el contexto correcto; Argo CD CLI instalado.
- Ejecutar los comandos desde la raíz del repositorio, donde están `argocd/` y `bootstrap/root.yaml`.
- Acceso a Internet para descargar imágenes y recursos; disponer de una terminal adicional para el port-forward.
- La CRD y el recurso `Database` deben existir antes de ejecutar el paso 4.

```bash
# 1. Argo CD
kubectl create namespace argocd
kubectl apply -n argocd --server-side --force-conflicts -k argocd
kubectl -n argocd rollout status statefulset/argocd-application-controller
kubectl -n argocd rollout status deploy/argocd-server

# 2. Acceso
kubectl -n argocd port-forward svc/argocd-server 8080:443     # otra terminal
argocd admin initial-password -n argocd
argocd login localhost:8080 --username admin --insecure
argocd account update-password
kubectl -n argocd delete secret argocd-initial-admin-secret

# 3. Bootstrap: un solo comando trae TODO lo demás
kubectl apply -f bootstrap/root.yaml

# 4. Dependencias externas (en el lab, el "operador" eres tú)
kubectl -n waves-demo patch database main-db --subresource=status --type merge -p '{"status":{"phase":"Ready"}}'

# Verificacion:
argocd app list