# 🏛️ Arquitectura de Referencia - MercaAndes (v1.0)

## 1. Visión General del Sistema
MercaAndes es una plataforma de datos omnicanal diseñada para procesar datos históricos de Olist (F1), transacciones simuladas de POS (F2), inventarios vía API (F3) y eventos en tiempo real de Marketplace (F4).

```text
  [F1 Olist]       [F2 POS]       [F3 API Inv.]    [F4 Marketplace]
      │               │                 │                 │
      ▼               ▼                 ▼                 ▼
  [Batch Load]   [DB Seeds]      [REST Consumer]   [RabbitMQ Producer]
      │               │                 │                 │
      └───────────────┴────────┬────────┴─────────────────┘
                               ▼
                        [S3 Data Lake] ──► [Spark ETL] ──► [RDS Postgres] ──► [FastAPI / Power BI]
```

## 2. Capas de Datos
- Raw / Bronze: Archivos CSV e ingestas crudas en S3.
- Silver: Datos limpios, deduplicados y seudonimizados (cumpliendo RN-11).
- Gold: Modelo Dimensional (Estrella) en RDS PostgreSQL para consumo de FastAPI y Power BI.

