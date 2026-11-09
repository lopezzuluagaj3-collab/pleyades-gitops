# 📊 Módulo: Stack SRE & Observabilidad Cloud-Native

Aprovisionamiento de la suite de confiabilidad **SRE (Site Reliability Engineering)** en el namespace `monitoring` mediante el operador oficial `kube-prometheus-stack` (v68.4.4) orquestado vía GitOps en ArgoCD.

---

## 🏛️ Arquitectura de Observabilidad y Principios SRE

### 1. El Modelo "Cerebro Central vs Sensores Distribuidos"
Para permitir la observabilidad elástica durante autoescalados en Microsoft Azure, el stack desacopla la recolección en dos planos:

```text
       [ Nodo 1 (Servidor Principal) ]                    [ Nodo 2 (Nuevo Nodo por Autoescalado) ]
 ┌────────────────────────────────────────┐             ┌────────────────────────────────────────┐
 │  Prometheus TSDB (Cerebro Central)    │             │  Pods de Microservicios                │
 │  Grafana (Panel Visual Unificado)      │             │                                        │
 │  node-exporter (Sensor DaemonSet)      │◄──Cilium───►│  node-exporter (Sensor DaemonSet)      │
 └────────────────────────────────────────┘     eBPF    └────────────────────────────────────────┘
```

* **Sensores de Hardware (`node-exporter`):** Desplegado como **`DaemonSet`**. Cada vez que Azure enciende un nuevo nodo por un pico de concurrencia, Kubernetes instala automáticamente una réplica ligera (~15 MiB RAM) para medir CPU, RAM, IOPS de disco y red en ese servidor.
* **Cerebro Central (`Prometheus`):** Instancia única con *Service Discovery* que descubre dinámicamente nuevos nodos y hace *scrape* de sus métricas a través del plano de datos eBPF de Cilium.
* **Panel Unificado (`Grafana`):** Instancia única que centraliza dashboards de toda la infraestructura.

---

## 🎯 Los 4 Golden Signals de SRE (Google SRE Framework)

El stack está pre-configurado para medir las cuatro señales doradas de confiabilidad:
1. **Latencia (*Latency*):** Tiempo de respuesta por percentil ($p50$, $p90$, $p99$).
2. **Tráfico (*Traffic*):** Tasa de solicitudes por segundo (RPS) y peticiones HTTP concurrentes.
3. **Errores (*Errors*):** Tasa de respuestas fallidas (códigos HTTP 5xx / 4xx).
4. **Saturación (*Saturation*):** Uso relativo de CPU, memoria RAM, límites de cuota y espacio en disco.

---

## ⚙️ Decisiones Técnicas y Hardening de GitOps

* **Server-Side Apply (`ServerSideApply=true`):** Las CRDs del Prometheus Operator (`prometheuses`, `servicemonitors`, `podmonitors`) superan el límite de 256 KB de la anotación clásica `kubectl.kubernetes.io/last-applied-configuration`. Se activa Server-Side Apply para delegar la validación directamente a la API de Kubernetes sin saturar etcd.
* **Persistencia en Azure Disk (`managed-csi`):** Prometheus utiliza un volumen persistente de 5 GiB en Azure Managed Disk (Standard SSD) con retención de métricas fijada en 12 horas para optimizar el consumo de almacenamiento y memoria.
* **Control Plane AKS Limpio:** Se desactivan monitores de `etcd`, `kubeScheduler` y `kubeControllerManager` que en AKS están administrados por Microsoft fuera del alcance de los pods de usuario.

---

## 📋 Componentes del Módulo

| Componente | Tipo de Despliegue | Propósito | Consumo Promedio |
| :--- | :--- | :--- | :--- |
| **Prometheus Operator** | `Deployment` | Reconcilia CRDs (`ServiceMonitor`, `PrometheusRule`). | ~40 MiB |
| **Prometheus Server** | `StatefulSet` | TSDB para almacenamiento y consulta PromQL. | ~350 MiB |
| **Alertmanager** | `StatefulSet` | Deduplicación y silenciamiento de alertas SRE. | ~40 MiB |
| **Grafana** | `Deployment` | Visualización de dashboards y alertas operativas. | ~80 MiB |
| **Node Exporter** | `DaemonSet` | Sensor de métricas a nivel de sistema operativo y kernel. | ~15 MiB / nodo |
| **kube-state-metrics** | `Deployment` | Métricas del estado de objetos de Kubernetes (Pods, PVCs). | ~35 MiB |

---

## 🔑 Acceso Local a Grafana

Una vez desplegado y sincronizado por ArgoCD, puedes acceder a Grafana ejecutando:

```powershell
kubectl port-forward svc/monitoring-stack-grafana -n monitoring 3000:80
```
* **URL:** `http://localhost:3000`
* **Usuario:** `admin`
* **Contraseña:** `prom-operator`
