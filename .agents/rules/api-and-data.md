---
title: "Mantener contratos y cálculos financieros explícitos"
description: "Sigue los patrones FastAPI/Pydantic del backend, conserva la reproducibilidad de los datos simulados y aísla la lógica financiera para poder probarla."
scope: project
globs:
  - "backend/app/**/*.py"
  - "backend/tests/**/*.py"
  - "frontend/src/lib/**/*.ts"
alwaysApply: false
---

Al modificar la API o la lógica financiera:

1. Tipifica parámetros y retornos de endpoints. Usa `Literal` para opciones cerradas cuando corresponda; para respuestas estructuradas, define o reutiliza modelos Pydantic y decláralos en `response_model`.
2. Mantén el filtrado y las transformaciones en helpers del área de métricas en lugar de duplicar lógica entre handlers.
3. Conserva los resultados deterministas que dependen de `generate_mock_movements(seed=42)`. Si modificas la generación aleatoria, revisa el efecto de `random.seed` sobre el estado aleatorio global.
4. Implementa cada cálculo financiero como una función que reciba datos y devuelva el resultado. Añade pruebas directas del helper backend cuando sea aislable; para cálculos del cliente, cubre casos representativos y límites en `frontend/src/lib/*.test.ts`.

Estos patrones se observan en `backend/app/routes.py` (tipos, modelos Pydantic, `response_model`, `filter_movements`, `summarize_movements` y `build_top_categories`), `backend/tests/test_routes.py`, `frontend/src/lib/financial-utils.ts` y `frontend/src/lib/financial-utils.test.ts`. `/health` devuelve una respuesta simple y no requiere crear un modelo artificial.
