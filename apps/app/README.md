# 🌌 Pléyades App - GitOps Deployment Manifests

Este directorio contiene los manifiestos declarativos de Kubernetes gestionados por **ArgoCD** para el despliegue del microservicio core **`pleyades-app`**.

---

## 🏛️ Arquitectura del Despliegue

| Recurso | Tipo / API | Descripción |
| :--- | :--- | :--- |
| `namespace.yaml` | `Namespace` | Namespace aislado `pleyades-app` con enforcement de Pod Security Standard `restricted`. |
| `configmap.yaml` | `ConfigMap` | Variables de entorno no sensibles (`SERVER_PORT`, `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_SSLMODE`). |
| `secret.yaml` | `Secret` | Credenciales de base de datos (`DB_USER`, `DB_PASSWORD`) y llave criptográfica `JWT_SECRET`. |
| `deployment.yaml` | `Deployment` | Réplicas del contenedor Distroless con sondas `/healthz` y `/readyz`, usuario nonroot (UID 65532). |
| `service.yaml` | `Service` | Servicio `ClusterIP` interno en el puerto `8080`. |
| `servicemonitor.yaml` | `ServiceMonitor` | Integración nativa con **Prometheus Operator** para recolección de métricas en `/metrics`. |
| `kustomization.yaml` | `Kustomization` | Orquestación Kustomize para ArgoCD. |

---

## 🔄 Flujo de Actualización Automatizada (Promotion Pipeline)
La imagen del contenedor en `deployment.yaml` es actualizada automáticamente por el pipeline de CI/CD de `pleyades-app` mediante Pull Requests contra este repositorio cada vez que se aprueba una nueva versión en `main`.
