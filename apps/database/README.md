# 🗄️ Módulo: Base de Datos Master (In-Cluster PostgreSQL 16)

Instancia de PostgreSQL 16 desplegada bajo el patrón **`StatefulSet`** en el namespace `database`. Actúa como el almacén de datos central de la plataforma Pléyades para autenticación, gestión de inquilinos y metadatos operativos.

---

## 🏛️ Arquitectura Técnica y Decisiones de Diseño

### 1. ¿Por qué `StatefulSet` y no `Deployment`?
* **Identidad de Red Determinística:** El pod se nombra `postgres-0`. Su FQDN interno en el CoreDNS del clúster es:
  ```text
  postgres-0.postgres-service.database.svc.cluster.local:5432
  ```
* **Enlace Persistente de Almacenamiento:** Mediante `volumeClaimTemplates`, Kubernetes crea un PVC exclusivo (`postgres-data-postgres-0`). Si el pod `postgres-0` se reprograma en otro nodo físico, Kubernetes desconecta el disco de Azure y lo reconecta al mismo pod sin pérdida de información.

### 2. Almacenamiento Persistente en Azure (`managed-csi`)
* **Controlador CSI:** Utiliza el controlador oficial de Azure Disk (`disk.csi.azure.com`) con la clase de almacenamiento `managed-csi`.
* **Capacidad:** `5Gi` (Standard SSD) con política `WaitForFirstConsumer` y `allowVolumeExpansion: true`.
* **Prevención de Error `lost+found`:** Se define `PGDATA=/var/lib/postgresql/data/pgdata` para aislar los datos binarios de PostgreSQL del directorio raíz del sistema de archivos ext4.

---

## 📋 Componentes Desplegados

| Manifiesto | Tipo | Propósito |
| :--- | :--- | :--- |
| [`namespace.yaml`](./namespace.yaml) | `Namespace` | Namespace aislado `database` para la capa de persistencia. |
| [`postgres-configmap.yaml`](./postgres-configmap.yaml) | `ConfigMap` | Variables no sensibles (`PGDATA`, `POSTGRES_DB: pleyades_db`). |
| [`postgres-secret.yaml`](./postgres-secret.yaml) | `Secret` | Credenciales de acceso (`POSTGRES_USER`, `POSTGRES_PASSWORD`). |
| [`postgres-service.yaml`](./postgres-service.yaml) | `Service` | Enrutamiento interno ClusterIP en el puerto `5432`. |
| [`postgres-statefulset.yaml`](./postgres-statefulset.yaml) | `StatefulSet` | Contenedor `postgres:16-alpine` con PVC de 5 GiB. |
| [`kustomization.yaml`](./kustomization.yaml) | `Kustomization` | Agrupa y valida el empaquetado del módulo. |
