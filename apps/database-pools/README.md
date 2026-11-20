# 🌌 Pléyades - Catálogo Declarativo de Database Pools con Crossplane

Este directorio contiene las definiciones declarativas para el aprovisionamiento automatizado de instancias de **Azure Database for PostgreSQL Flexible Server (ADBFP)** gestionadas mediante **Crossplane** y **Workload Identity (OIDC)**.

---

## 🏛️ Arquitectura de Sharded Pools

Cada instancia declarada en este catálogo representa un servidor físico administrado que alberga un grupo de **5 a 7 bases de datos de inquilinos (tenants)** aisladas lógicamente:

* **Región:** `centralus` (sin restricciones de cuota para PostgreSQL Flexible Server).
* **Motor:** PostgreSQL 16 con TLS obligatorio (`require_secure_transport = on`).
* **SKU:** `Standard_B1ms` (1 vCPU, 2 GiB RAM, límite seguro de 50 conexiones concurrentes).
* **Almacenamiento:** 32 GiB con auto-crecimiento (*Auto-Grow*) activado.
* **Firewall:** Acceso interno seguro a través de la infraestructura troncal de Azure (`0.0.0.0` - `0.0.0.0`).
* **Secretos:** Generación automática de contraseña maestra y volcado directo a `Secret/psql-*-conn` en Kubernetes.

---

## ⚡ Ciclo de Vida Autónomo (Warm Pool Provisioning)

1. El microservicio `pleyades-app` monitorea la ocupación del pool activo mediante el catálogo `database_pools`.
2. Al alcanzar el **umbral de advertencia (5 de 7 inquilinos ocupados)**, se dispara la solicitud del siguiente servidor (`psql-pleyades-dev-02`).
3. Crossplane asume la identidad `uami-crossplane-pleyades` y orquesta la creación en Azure ARM.
4. Al completarse el aprovisionamiento (5-8 minutos), Crossplane emite el `Secret` de conexión.
5. El microservicio detecta el nuevo servidor y lo auto-registra en la tabla `database_pools` con estado `ACTIVE`.
6. Cuando el servidor actual se llena con el inquilino 7 (conmutando a `FULL`), los nuevos inquilinos son alojados transparentemente en el nuevo servidor.
