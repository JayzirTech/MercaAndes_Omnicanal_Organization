# 🗺️ Mapa Maestro de MercaAndes
> **Nuestra hoja de ruta y guía de navegación del proyecto.**

El proyecto busca construir una solución completa de datos para **MercaAndes**, una plataforma de marketplace/e-commerce que integra datos históricos de Olist con datos simulados de operación, los procesa mediante pipelines de datos y los expone para análisis y consumo mediante APIs y Power BI.

---

## 🎯 Las 4 Capas del Mapa
Para evitar aprender tecnologías de forma aislada o perder la trazabilidad del proyecto, estructuramos este mapa en cuatro capas continuas:
1. **Qué exige el proyecto** (Requisitos del enunciado)
2. **Qué tenemos que construir** (Implementación técnica / código)
3. **Qué conocimiento necesitamos adquirir** (Proceso de aprendizaje)
4. **Qué evidencia / documentación debemos producir** (Entregables)

---

## 🏛️ Visión General de Arquitectura (Hipótesis Inicial)

```
                         MERCAANDES
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
   Datos históricos      Datos simulados       Eventos
      Olist             POS / Inventario      Marketplace
        │                     │                     │
        └─────────────────────┼─────────────────────┘
                              ▼
                       INGESTA / MENSAJERÍA
                              │
                    ┌─────────┴─────────┐
                    │                   │
                    ▼                   ▼
                 Batch              Eventos
                    │                   │
                    └─────────┬─────────┘
                              ▼
                         PROCESAMIENTO
                              │
                           Spark
                              │
                              ▼
                       ALMACENAMIENTO
                              │
                    ┌─────────┴─────────┐
                    │                   │
                    ▼                   ▼
                   S3                  RDS
                    │                   │
                    └─────────┬─────────┘
                              ▼
                        CONSUMO / APIs
                              │
                    ┌─────────┴─────────┐
                    │                   │
                    ▼                   ▼
                 FastAPI             Power BI
                    │
                    ▼
              Usuarios / Clientes
```

> **Nota:** Este diagrama representa nuestra *hipótesis inicial de arquitectura*. A medida que iteremos y aprendamos las tecnologías, iremos tomando decisiones documentadas de diseño (ADRs).

---

## 1. 🧩 Los 20 Temas del Proyecto

| # | Tema | ¿Para qué lo necesitamos? | Evidencia / Entregable |
|---|---|---|---|
| 1 | **FastAPI** | APIs de negocio | API funcionando con OpenAPI / Swagger |
| 2 | **Docker** | Contenerizar servicios | `Dockerfile` / `docker-compose.yml` |
| 3 | **AWS** | Infraestructura cloud | Recursos provisionados |
| 4 | **VPC / Subredes** | Red y seguridad | Diagrama de red y subredes públicas/privadas |
| 5 | **RabbitMQ** | Mensajería de eventos | Productor/Consumidor + Queues/Exchanges |
| 6 | **Airflow** | Orquestación batch | DAGs de orquestación ejecutándose |
| 7 | **Celery** | Ejecución distribuida | Workers y colas de procesamiento de tareas |
| 8 | **Spark** | Procesamiento masivo de datos | Jobs de Spark ETL/ELT |
| 9 | **S3** | Data Lake / Almacenamiento objeto | Buckets estructurados (Raw, Bronze, Silver, Gold) |
| 10 | **RDS** | Base de datos relacional | PostgreSQL desplegado y poblado |
| 11 | **ECS / Fargate** | Ejecución de contenedores en la nube | Servicios desplegados y saludables |
| 12 | **ALB** | Entrada HTTP y balanceo | Load Balancer con target groups |
| 13 | **EKS / Kubernetes** | Orquestación avanzada de contenedores | Cluster funcional / Manifestos K8s |
| 14 | **CI/CD** | Automatización de integración y entrega | Pipeline de GitHub Actions ejecutado |
| 15 | **Seguridad** | Protección de accesos y credenciales | Roles IAM + Secrets Manager |
| 16 | **Performance** | Evaluación de capacidad del sistema | Reporte de Benchmarks y métricas |
| 17 | **Observabilidad** | Monitoreo y diagnóstico | Centralización de Logs y Métricas |
| 18 | **Power BI** | Capa analítica y visualización | Dashboard con al menos 5 KPIs interactivos |
| 19 | **Costos AWS** | Control y optimización financiera | Estimación mensual y tagging |
| 20 | **Documentación** | Trazabilidad del proyecto | README, ADRs, Runbooks y Bitácora |

---

## 2. 📦 Fuentes de Datos

### **Fuente F1 — Olist (Histórico)**
Dataset del e-commerce brasileño (~1.5M de filas en 9 CSVs):
* `customers`, `orders`, `order_items`, `products`, `sellers`, `payments`, `reviews`, `geolocation`, `category_translation`.

### **Fuente F2 — POS Simulator (Simulado)**
Simulador de ventas físicas/tiendas (mínimo 1M de transacciones):
* Debe simular: anulaciones, devoluciones, descuentos, múltiples medios de pago y ventas multi-tienda.

### **Fuente F3 — API Inventario (Simulado)**
API de stock, SKUs, sellers y disponibilidad:
* Debe fallar intencionalmente generando errores HTTP 429 y 503 en al menos un **3% de las solicitudes**.

### **Fuente F4 — Marketplace (Generador de Eventos)**
Emisor de eventos de negocio (`order_created`, `payment_confirmed`, `shipped`, `delivered`):
* Debe introducir escenarios complejos: **duplicados**, **eventos fuera de orden** y **eventos tardíos**.

---

## 3. 🔄 Pipeline de Datos

```
             FUENTES
                │
       ┌────────┼────────┐
       │        │        │
      F1       F2       F3/F4
    Olist      POS    APIs/Eventos
       │        │        │
       └────────┼────────┘
                ▼
             INGESTA
                │
        ┌───────┴────────┐
        │                │
       Batch           Eventos
        │                │
        ▼                ▼
       S3             RabbitMQ
        │                │
        └───────┬────────┘
                ▼
             Spark
                │
                ▼
        Modelo Dimensional
                │
                ▼
               RDS
                │
        ┌───────┴────────┐
        ▼                ▼
     FastAPI          Power BI
```

---

## 4. 📊 Modelo de Datos Dimensional

Transformaremos los datos crudos en un modelo de **Hechos y Dimensiones**:

```
                 dim_customer
                      │
                      │
dim_date ───── fact_orders ───── dim_product
                      │
                      │
                 dim_seller
                      │
                 dim_location
```

**Criterios del modelo:**
* Claves sustitutas (*surrogate keys*).
* Integridad referencial explícita.
* Definición clara de granularidad por tabla.
* Reglas de calidad de datos y trazabilidad.

---

## 5. 📨 Reglas de Mensajería y Eventos

El sistema de eventos (RabbitMQ + Celery + Spark) debe demostrar resistencia a fallos implementando:
* **ACK** (Ause de recibo explícito).
* **Estrategias de Reintentos** (*Retries*).
* **DLQ** (*Dead Letter Queue* para mensajes fallidos).
* **Idempotencia** (manejo seguro de duplicados).
* Procesamiento correcto de **eventos tardíos** y **fuera de orden**.

---

## 6. 🔐 Seguridad, CI/CD y Performance

* **Seguridad:** Gestión de credenciales vía Secrets Manager, principio de menor privilegio con IAM, aislamientos por VPC/Subnets/Security Groups.
* **CI/CD:** Automatización mediante GitHub Actions cubriendo `Test` ➔ `Build` ➔ `Deploy` ➔ `Rollback`.
* **Performance:** Pruebas de estrés y latencia con reportes de métricas "Antes vs Después".

---

## 7. 📚 Estructura de Documentación Técnicas

```text
docs/
├── architecture/
│   ├── architecture.md
│   └── diagrams/
├── adr/
│   ├── ADR-001-seleccion-framework-backend.md
│   └── ...
├── bitacora/
│   ├── 2026-09-24-inicio-sprint-1.md
│   └── ...
├── api/
├── data/
├── aws/
├── deployment/
├── testing/
├── performance/
└── runbook/
```

### 📝 Estructura Obligatoria de Bitácora
Cada vez que aprendamos un tema o tomemos una decisión técnica, registraremos una entrada con la siguiente estructura:
1. Fecha + Tema
2. ¿Qué aprendimos?
3. ¿Por qué lo necesitamos?
4. ¿Cómo funciona?
5. ¿Cómo lo aplicamos a MercaAndes?
6. Implementación
7. Problemas encontrados
8. Solución
9. Decisiones tomadas
10. Evidencias
11. Pendientes

---

## 🧠 El Ciclo de Aprendizaje e Implementación

```
                    MERCAANDES
                        │
          ┌─────────────┴─────────────┐
          │                           │
       APRENDER                   IMPLEMENTAR
          │                           │
          ▼                           ▼
       Concepto                    Código
          │                           │
          └─────────────┬─────────────┘
                        ▼
                      PROBAR
                        │
                        ▼
                   DOCUMENTAR
                        │
                        ▼
                     GIT
                        │
                        ▼
                     JIRA
                        │
                        ▼
                  EVIDENCIA FINAL
```

---

## 🗂️ Mapeo a Epics en Jira

* **EPIC 1** — Fundamentos y arquitectura
* **EPIC 2** — Datos y simuladores
* **EPIC 3** — Backend y APIs
* **EPIC 4** — Dockerización
* **EPIC 5** — AWS e infraestructura
* **EPIC 6** — Mensajería
* **EPIC 7** — Orquestación y procesamiento
* **EPIC 8** — Modelo de datos
* **EPIC 9** — CI/CD
* **EPIC 10** — Kubernetes
* **EPIC 11** — Observabilidad y performance
* **EPIC 12** — Power BI
* **EPIC 13** — Costos
* **EPIC 14** — Documentación y entrega

---

## 🔗 Matriz de Control de Requisitos

| Requisito | Tecnología | Implementación | Jira | Git | Documentación | Evidencia |
|---|---|:---:|:---:|:---:|:---:|:---:|
| API | FastAPI | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |
| Contenedores | Docker | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |
| Cloud | AWS | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |
| Mensajería | RabbitMQ | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |
| Orquestación | Airflow | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |
| Distribución | Celery | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |
| Procesamiento | Spark | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |
| Data Lake | S3 | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |
| Base de datos | RDS PostgreSQL | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |
| Contenedores Cloud | ECS / Fargate | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |
| Balanceador | ALB | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |
| Kubernetes | EKS | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |
| Automatización | CI/CD | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |
| BI | Power BI | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |
| Seguridad | IAM / Secrets | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |
| Performance | Benchmark | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |
| Costos | AWS Cost | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ |

---