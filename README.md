# Andén en Kubernetes

Manifiestos de estudio para un clúster Amazon EKS. No hay un clúster en esta máquina y no se aplica nada: un EKS real cobra por el control plane y por los nodos.

Cada tema del temario está en un archivo concreto.

| Tema | Dónde |
| --- | --- |
| Arquitectura y conceptos | este README y `eks/cluster.yaml` |
| Cluster EKS con eksctl | `eks/cluster.yaml` |
| kubectl, AWS CLI, eksctl | comandos de abajo |
| Pods, Services y YAML | `manifests/02-pod.yaml`, `manifests/03-services.yaml` |
| ClusterIP, NodePort, LoadBalancer | `manifests/03-services.yaml` |
| Namespaces, labels y selectors | `manifests/00-namespace.yaml` y el `selector` de cada workload |
| ReplicaSets y Deployments | `manifests/04-replicaset.yaml`, `manifests/05-deployment.yaml` |
| Rolling update y rollback | `strategy.rollingUpdate` en el Deployment `mostrador-roll` |
| Pods con sidecar | `manifests/06-sidecar.yaml` |
| DaemonSets y StatefulSets | `manifests/07-daemonset.yaml`, `manifests/08-statefulset.yaml` |
| PV y PVC | `manifests/09-storage.yaml` |
| Almacenamiento con EBS | StorageClass `ebs-gp3` y el volumen CSI |
| ConfigMaps, Secrets y variables | `manifests/01-config.yaml` y el Pod `puesto` |
| HPA y ResourceQuota | `manifests/10-escala.yaml` |
| Jobs y CronJobs | `manifests/11-jobs.yaml` |
| Helm | `helm/anden` |
| GitOps con Argo CD | `gitops/application.yaml` |
| Prometheus y Grafana | `observability/prometheus-grafana.yaml` |
| Blue-green | `blue-green/mostrador.yaml` |

## Validar sin clúster

```bash
kubeconform -summary -kubernetes-version 1.32.0 -ignore-missing-schemas \
  projects/k8s-anden/manifests projects/k8s-anden/blue-green projects/k8s-anden/observability projects/k8s-anden/gitops
helm lint projects/k8s-anden/helm/anden
helm template mostrador projects/k8s-anden/helm/anden | kubeconform -summary -kubernetes-version 1.32.0
```

`gitops/application.yaml` es un recurso de Argo CD. kubeconform lo deja pasar con `-ignore-missing-schemas` porque el esquema no viene en la instalación local.

## Si algún día hay cuenta de AWS

```bash
eksctl create cluster -f projects/k8s-anden/eks/cluster.yaml
kubectl apply -f projects/k8s-anden/manifests
kubectl -n anden rollout status deploy/mostrador-roll
kubectl -n anden rollout undo deploy/mostrador-roll
kubectl -n anden patch svc mostrador --type merge -p '{"spec":{"selector":{"color":"green"}}}'
```

El Service `mostrador` apunta a `color: blue`. El corte a verde es cambiar ese selector. `eksctl create cluster` crea nodos de pago: bórralo con `eksctl delete cluster -f projects/k8s-anden/eks/cluster.yaml` cuando termines.
