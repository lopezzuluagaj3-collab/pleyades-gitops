# ⚙️ Módulo: Crossplane (Universal Cloud Control Plane)

Permite aprovisionar recursos cloud de Microsoft Azure directamente mediante manifiestos YAML nativos de Kubernetes, utilizando autenticación federada OIDC (sin secretos estáticos).

---

## 📋 Componentes
- **Operador Crossplane:** Core controller en el namespace `crossplane-system`.
- **Azure Provider:** `provider-azure` (Upbound) para gestionar Azure Blob Storage y recursos cloud.
- **ProviderConfig:** Configuración de credenciales federadas con la identidad administrada de Azure.
- **Compositions & XRDs:** Plantillas declarativas de infraestructura para desarrolladores.
