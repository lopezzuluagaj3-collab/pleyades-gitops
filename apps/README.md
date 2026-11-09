# 📂 Catálogo de Aplicaciones de Plataforma (`apps/`)

Este directorio contiene las definiciones de los componentes y controladores que componen la plataforma **Pléyades**, gestionados automáticamente por la aplicación raíz de ArgoCD (`bootstrap/root-app.yaml`) bajo el patrón **App-of-Apps**.

---

## 🏛️ Componentes Planificados

1. **`crossplane/`**:
   - Motor de *Infrastructure as Code* declarativo en Kubernetes.
   - Provider oficial de Microsoft Azure autenticado vía **Workload Identity (OIDC)**.
   - Definiciones de Recursos Compuestos (`XRDs` y `Compositions`) para exponer abstracciones de almacenamiento (*Azure Blob Storage*) y bases de datos a los desarrolladores.

2. **`database/`**:
   - Despliegue de **PostgreSQL StatefulSet** optimizado para desarrollo en el clúster.
   - Almacenamiento persistente con **PersistentVolumeClaim (PVC)** respaldado por Azure Managed Disks.
   - Base de datos Master para catálogo de usuarios y enrutamiento multi-tenant.

3. **`monitoring/`**:
   - Stack completo de confiabilidad **SRE**: `kube-prometheus-stack` (Prometheus Operator, Alertmanager y Grafana).
   - Definición formal de **SLIs, SLOs y Error Budgets** con alertas accionables basadas en *Burn Rate*.
   - Dashboards de los *4 Golden Signals* (Latencia, Tráfico, Errores y Saturación).
