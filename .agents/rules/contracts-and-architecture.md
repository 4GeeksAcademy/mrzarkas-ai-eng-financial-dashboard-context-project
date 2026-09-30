---
title: "Conservar los contratos y la arquitectura del dashboard"
description: "Mantén coordinados los modelos frontend/backend y respeta la organización demostrada por el repositorio al ampliar el dashboard."
scope: project
globs:
	- "backend/app/**/*.py"
	- "frontend/src/**/*.ts"
	- "frontend/src/**/*.tsx"
alwaysApply: false
---

Antes de modificar contratos o estructura:

1. Si cambias `FinancialMovement` en `backend/app/routes.py`, revisa y actualiza su equivalente en `frontend/src/lib/financial-types.ts`, junto con consumidores y pruebas relevantes. Conserva los nombres serializados actuales salvo que coordines explícitamente un cambio en ambos lados.
2. Mantén endpoints, modelos y lógica de métricas en `backend/app/`; la composición de la interfaz en `frontend/src/App.tsx`; los componentes visuales en `frontend/src/components/`; y los tipos y utilidades del cliente en `frontend/src/lib/`.
3. No asumas que existe una base de datos, almacenamiento persistente o cliente API generado. Los movimientos actuales se simulan en memoria. Si una tarea requiere otra arquitectura, descríbela como un cambio explícito, no como una convención ya existente.
4. Si cambias el rango temporal de los datos, haz que la etiqueta del periodo represente ese rango o deja claro que es un título fijo. No afirmes un periodo que los datos presentados no respalden.

La base de estas instrucciones está en `backend/app/main.py`, `backend/app/routes.py`, `frontend/src/App.tsx`, `frontend/src/lib/financial-types.ts`, `frontend/src/components/` y `docker-compose.yml`. El modelo `FinancialMovement` se declara por separado en Python y TypeScript; actualmente incluye `create_date`, `amount`, `operation_type`, `category` y `business_type`. La etiqueta `2024 - Full Year` es fija mientras el generador selecciona el año según la fecha actual.
