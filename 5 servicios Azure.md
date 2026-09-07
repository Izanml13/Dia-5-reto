# Guía de decisión: servicios Azure

Referencia rápida de cinco servicios de Azure: qué resuelve cada uno, cuándo encaja y cuándo no.

| # | Servicio | Categoría | En una frase |
|---|---|---|---|
| 1 | Kubernetes (AKS) | Contenedores / IaaS-PaaS | Orquestación con control total |
| 2 | App Service | PaaS de aplicaciones | Web/API sin gestionar infraestructura |
| 3 | Azure SQL Database | Base de datos relacional | SQL Server gestionado, ACID |
| 4 | Cosmos DB | Base de datos NoSQL | Distribución global, baja latencia |
| 5 | Microsoft Foundry | Plataforma de IA | GenAI, RAG y agentes |

---

## 1. Kubernetes (AKS en Azure)

**Qué es:** orquestador de contenedores. Se usa cuando necesitas control total sobre despliegue, red y escalado.

### ✅ Usar para

| Escenario | Detalle |
|---|---|
| **Microservicios** | Múltiples APIs/contenedores con despliegues independientes, service discovery y rolling updates |
| **Cargas elásticas / batch** | `HPA` + `Cluster Autoscaler` para picos; `Jobs` y `CronJobs` para procesos programados |
| **Portabilidad híbrida** | El mismo manifiesto en local, AKS y on-prem |
| **Aislamiento avanzado** | Namespaces, network policies, service mesh (Istio / Linkerd) |

### ❌ No usar para

Una sola web o API simple sin equipo de DevOps: queda sobredimensionado frente a App Service.

---

## 2. Azure App Service

**Qué es:** PaaS para webs y APIs sin gestionar infraestructura.

### ✅ Usar para

| Escenario | Detalle |
|---|---|
| **Web/API monolítica o ligera** | Por ejemplo, `get_completion()` de `recipe_summary.py:6` expuesto como API REST |
| **MVPs y backends simples** | Despliegue desde GitHub, slots staging/production, TLS y autoscale integrados |
| **Integración PaaS nativa** | Easy Auth (Entra ID), Key Vault references, VNET integration |
| **Tareas programadas ligeras** | WebJobs con Always On |

### ❌ No usar para

Contenedores stateful complejos, cargas con GPU o control de bajo nivel de red/kernel.

---

## 3. Azure SQL Database (SQL Dataservice)

**Qué es:** SQL relacional gestionado (PaaS), con compatibilidad T-SQL al 100 %.

### ✅ Usar para

| Escenario | Detalle |
|---|---|
| **Datos transaccionales/relacionales** | ERP, CRM, pedidos y facturación con ACID, JOINs y claves foráneas |
| **OLTP con consistencia fuerte** | Reporting mediante `Hyperscale` y read replicas |
| **Migración lift-and-shift** | Desde SQL Server on-prem, con modelo `DTU`/`vCore` y backup PITR automático |
| **Integridad y seguridad** | Always Encrypted, Row-Level Security, autenticación Microsoft Entra |

### ❌ No usar para

Documentos JSON puros, series temporales masivas o distribución global multimaestro de baja latencia. Para eso, Cosmos DB.

---

## 4. Azure Cosmos DB

**Qué es:** NoSQL globalmente distribuido y multi-modelo (NoSQL, MongoDB, Cassandra, Gremlin, Table).

### ✅ Usar para

| Escenario | Detalle |
|---|---|
| **Apps globales de baja latencia** | Catálogos, sesiones, carritos, IoT con replicación multi-región <10 ms |
| **Datos semiestructurados / alta ingesta** | Telemetría, chat, perfiles de usuario donde el esquema cambia |
| **Escalado elástico por `RU/s`** | Picos impredecibles con autoscale y TTL para retención automática |
| **Alta disponibilidad** | 99,999 % con configuración multimaestro |

### ❌ No usar para

Transaccional relacional complejo con JOINs, analytics pesado ad-hoc (mejor Synapse / Fabric) o escenarios sensibles al coste con bajo volumen.

---

## 5. Microsoft Foundry (antes AI Studio)

**Qué es:** plataforma para construir, evaluar y operar aplicaciones de IA generativa y agentes sobre Azure OpenAI y modelos abiertos.

### ✅ Usar para

| Escenario | Detalle |
|---|---|
| **RAG empresarial** | Indexado de documentos en AI Search + grounding con prompts controlados (`recipe_summary.py:34-35`), con evaluación y guardrails |
| **Agentes y orquestación** | Function calling, entornos y despliegue de `gpt-4o`, Llama o Mistral con observabilidad |
| **MLOps / LLMOps** | Prompt flow, evaluaciones de calidad y seguridad, content filters, despliegue a AKS o App Service como endpoint |
| **Integración de tu caso** | Sustituir `OpenAI(api_key=...)` (`:4`) por inferencia gestionada en Foundry con Managed Identity + Content Safety |

### ❌ No usar para

Hosting de aplicaciones web generales, como base de datos, o para entrenar un modelo foundation desde cero.

---

## Resumen: decisión rápida

| Necesidad | Elegir |
|---|---|
| Contenedores, microservicios | **Kubernetes / AKS** |
| 1 Web/API rápida sin Ops | **App Service** |
| SQL relacional ACID | **Azure SQL Database** |
| Global, NoSQL, <10 ms | **Cosmos DB** |
| IA generativa, RAG, agentes | **Microsoft Foundry** |

### Cómo encajan entre sí

No son alternativas excluyentes, sino capas distintas de una misma arquitectura:

- **Cómputo:** App Service (simple) o AKS (control total)
- **Datos:** Azure SQL (relacional) o Cosmos DB (global, NoSQL)
- **IA:** Foundry como capa de inferencia, consumida desde el cómputo mediante Managed Identity
