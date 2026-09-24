# ADR-001: Estructura Base, Flujo Git y Convenciones del Proyecto

- **Estado**: Aprobado
- **Fecha**: 2026-09-24
- **Autores**: Equipo MercaAndes

## Contexto
El proyecto exige la participación coordinada del equipo trabajando sobre 20 áreas técnicas en 6 semanas. Para evitar conflictos de código y mantener la trazabilidad de requerimientos, se establece el flujo de trabajo estándar.

## Decisión
1. Adoptar el modelo de GitFlow simplificado (`main` para producción, `dev` para integración, ramas `feature/*`, `fix/*`, `chore/*`, `docs/*`).
2. Documentar cada hito mediante la plantilla obligatoria de Bitácora (`docs/bitacora/`).
3. Registrar decisiones de arquitectura críticas mediante ADRs.

## Consecuencias
- **Positivas**: Garantiza revisiones estructuradas, previene código roto en producción y mantiene trazabilidad para la evaluación final.
- **Negativas**: Requiere disciplina en el flujo de commits y PRs.