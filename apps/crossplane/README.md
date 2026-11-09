# ⚙️ Módulo: Crossplane (Universal Cloud Control Plane)

Permite aprovisionar recursos cloud de Microsoft Azure directamente mediante manifiestos YAML nativos de Kubernetes, utilizando **Azure Workload Identity (OIDC federated credentials)** sin almacenamiento de secretos estáticos (Zero Static Secrets).

---

## 🏛️ Arquitectura de Autenticación Federada (Workload Identity)

En lugar de utilizar secretos estáticos (`client_secret` de Service Principals) almacenados en Secrets de Kubernetes —lo cual representa una vulnerabilidad crítica ante fugas y rotación manual—, este módulo utiliza **OIDC Token Exchange**:

1. **AKS OIDC Issuer:** AKS expone una URL OIDC pública de emisor de tokens criptográficos (`/.well-known/openid-configuration`).
2. **Kubernetes ServiceAccount:** Se anota la cuenta de servicio `provider-azure-storage` con el `client-id` de la Managed Identity (`uami-crossplane-pleyades`).
3. **Azure Federated Identity Credential:** Microsoft Entra ID valida la firma del token emitido por el ServiceAccount de AKS con audiencia `api://AzureADTokenExchange`.
4. **Token In-Memory:** El webhook de AKS inyecta un token temporal proyectado en `/var/run/secrets/azure/tokens/azure-identity-token`, renovado de manera autónoma sin intervención humana.

> **💡 Equivalencia Cloud:** En AWS, este patrón corresponde exactamente a **AWS IRSA (IAM Roles for Service Accounts)** o **EKS Pod Identity**, donde el ServiceAccount de Kubernetes asume un rol IAM mediante OIDC federation.

---

## 📦 Componentes Desplegados

| Manifiesto | Tipo | Propósito |
| :--- | :--- | :--- |
| [`deployment-runtime-config.yaml`](./deployment-runtime-config.yaml) | `DeploymentRuntimeConfig` | Personaliza el Deployment del proveedor, fijando el nombre del ServiceAccount y configurando la mutación de Workload Identity y límites de CPU/RAM. |
| [`provider-family-azure.yaml`](./provider-family-azure.yaml) | `Provider` | Paquete base de Upbound (`provider-family-azure`) que gestiona el runtime y tipos comunes. |
| [`provider-azure-storage.yaml`](./provider-azure-storage.yaml) | `Provider` | Proveedor granular Upbound especializado en Azure Storage (`xpkg.upbound.io/upbound/provider-azure-storage`). |
| [`provider-config.yaml`](./provider-config.yaml) | `ProviderConfig` | Vincula el proveedor con el método de credenciales `OIDCTokenFile` y las identidades de Azure. |
| [`kustomization.yaml`](./kustomization.yaml) | `Kustomization` | Agrupa y orquesta el orden de aplicación de los recursos. |
