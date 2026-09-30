---
title: "Seguir las convenciones de naming de cada stack"
description: "Usa los estilos de nombres ya establecidos para Python y TypeScript/React y conserva el kebab-case en archivos de componentes."
scope: project
globs:
  - "backend/**/*.py"
  - "frontend/src/**/*.ts"
  - "frontend/src/**/*.tsx"
alwaysApply: false
---

Al añadir o renombrar símbolos y archivos de código:

1. En Python, usa `snake_case` para módulos, funciones y variables; usa `PascalCase` para clases y modelos.
2. En TypeScript/React, usa `PascalCase` para componentes y tipos; usa `camelCase` para funciones y variables.
3. Mantén los archivos de componentes React en `kebab-case`. Para otros módulos frontend, conserva el estilo de nombres que ya usa el área correspondiente.
4. Al renombrar un símbolo o archivo, actualiza sus imports, referencias y pruebas afectadas.

La convención se basa en ejemplos existentes: `generate_mock_movements` y `FinancialMovement` en `backend/app/routes.py`; `computeMonthlyData` en `frontend/src/lib/financial-utils.ts`; `FinancialMovement` en `frontend/src/lib/financial-types.ts`; y el componente `KPIRow` en `frontend/src/components/dashboard/kpi-row.tsx`.
