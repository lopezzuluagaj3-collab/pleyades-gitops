# 🌌 Proyecto Pléyades — Platform GitOps & SRE Control Plane

[![GitOps: ArgoCD](https://img.shields.io/badge/GitOps-ArgoCD_Pull--Based-EF7B42?logo=argo&logoColor=white)](https://argoproj.github.io/cd/)
[![Crossplane](https://img.shields.io/badge/Crossplane-Cloud_Control_Plane-blue?logo=crossplane&logoColor=white)](https://crossplane.io/)
[![Observability: Prometheus](https://img.shields.io/badge/SRE-Prometheus_SLOs-E6522C?logo=prometheus&logoColor=white)](https://prometheus.io/)
[![Grafana](https://img.shields.io/badge/Visualization-Grafana_Golden_Signals-F46800?logo=grafana&logoColor=white)](https://grafana.com/)
[![Git Strategy](https://img.shields.io/badge/Git_Flow-Trunk--Based_Development-blueviolet)](https://trunkbaseddevelopment.com/)

El cerebro declarativo de la plataforma **Pléyades**. Todo lo que existe y opera dentro del clúster de Kubernetes (**AKS**) se define aquí mediante manifiestos YAML puros. **ArgoCD** reconcilia continuamente el estado deseado contra el clúster vivo en Microsoft Azure.

---

## 🏛️ Arquitectura del Repositorio (Patrón App-of-Apps)

Este repositorio implementa el estándar **App-of-Apps** de ArgoCD:

```mermaid
flowchart TD
    subgraph GitRepository["Repositorio: pleyades-gitops (Git)"]
        RootAppYAML["bootstrap/root-app.yaml (App Madre)"]
        
        subgraph AppsFolder["apps/ (Directorio Monitoreado)"]
            CrossplaneDef["apps/crossplane/"]
            DatabaseDef["apps/database/"]
            MonitoringDef["apps/monitoring/"]
        end
    end

    subgraph AKSCluster["Clúster AKS (Azure)"]
        ArgoServer["ArgoCD Server / Controller"]
        
        subgraph PlatformPods["Plataforma en Ejecución"]
            CrossplaneOperator["Crossplane (Azure Provider)"]
            PostgresMaster["PostgreSQL StatefulSet (Master DB)"]
            SREStack["Prometheus + Alertmanager + Grafana"]
        end
    end

    RootAppYAML -->|"1. Conecta con ArgoCD"| ArgoServer
    ArgoServer -->|"2. Vigila continuamente 'apps/'"| AppsFolder
    AppsFolder -->|"3. Auto-sincroniza (Pull-Based)"| PlatformPods
```

---

## 📂 Estructura del Proyecto

```text
pleyades-gitops/
├── bootstrap/                    # Aplicación Raíz que gobierna todo el clúster
│   └── root-app.yaml             # Definición de la Application 'pleyades-root-app'
│
├── apps/                         # Catálogo de componentes gestionados por ArgoCD
│   ├── crossplane/               # Crossplane Operator, Provider Azure y Compositions
│   ├── database/                 # PostgreSQL StatefulSet (Master DB para desarrollo)
│   └── monitoring/               # Stack SRE: Prometheus, Grafana, SLOs y Error Budgets
│
└── README.md                     # Documentación del plano de control GitOps
```

---

## 🚀 ¿Cómo se Inicializa en el Clúster?

Una vez desplegada la infraestructura base con Terraform, solo se requiere presentar la aplicación raíz a ArgoCD una única vez:

```powershell
kubectl apply -f bootstrap/root-app.yaml
```

A partir de ese instante, **ArgoCD asume el control absoluto**: cualquier cambio, adición o eliminación de archivos dentro del directorio `apps/` en la rama `main` se sincroniza automáticamente en el clúster sin intervención humana.

---

<p align="center">
  <b>Proyecto Pléyades</b> • Diseñado y construido por <b>Juan Diego López Zuluaga</b><br/>
  <i>DevOps, Cloud Platform & SRE Engineer</i>
</p>
