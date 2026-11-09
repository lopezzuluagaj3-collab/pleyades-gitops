# 🗄️ Módulo: Base de Datos Master (In-Cluster PostgreSQL)

PostgreSQL desplegado como `StatefulSet` en el namespace `database` para actuar como base de datos de soporte del clúster (auth, catálogo de inquilinos y metadata de enrutamiento).

---

## 📋 Componentes
- **StatefulSet:** Instancia de PostgreSQL con almacenamiento persistente (`azure-managed-disk` PVC).
- **Service (Headless / ClusterIP):** Enrutamiento interno para microservicios.
- **ConfigMap & Secret:** Variables de entorno y credenciales iniciales.
