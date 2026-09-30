---
title: "Validar cambios con las herramientas existentes"
description: "Añade pruebas al nivel adecuado y ejecuta e informa las validaciones del stack afectado sin atribuir resultados a comandos no ejecutados."
scope: project
globs:
  - "backend/**/*.py"
  - "frontend/src/**/*.ts"
  - "frontend/src/**/*.tsx"
  - "frontend/package.json"
  - "backend/requirements.txt"
alwaysApply: false
---

Al cambiar código o comportamiento:

1. Para endpoints, filtros, agregaciones o respuestas de API, añade o actualiza pruebas en `backend/tests/`. Usa `TestClient(app)` para comportamiento HTTP y prueba directamente helpers cuando la lógica sea aislable. Comprueba filtros, campos y orden relevantes; ejecuta `pytest` desde `backend/`.
2. Para cálculos o formatos del cliente, añade o actualiza pruebas `.test.ts` junto a los helpers en `frontend/src/lib/`; ejecuta `npm test` desde `frontend/`. No presentes esta suite como cobertura de componentes o integración API si no existen pruebas para esos comportamientos.
3. Según el cambio, ejecuta en `frontend/` `npm test`, `npm run lint` y/o `npm run build`; para el backend ejecuta `pytest` en `backend/`.
4. Informa qué validaciones se ejecutaron y sus resultados. No afirmes que pasó un comando que no se ejecutó o que no pudo arrancar.

La estrategia existente se refleja en `backend/tests/test_routes.py`, `backend/tests/conftest.py`, `backend/requirements.txt`, `frontend/src/lib/financial-utils.test.ts` y los scripts de `frontend/package.json`. No hay un script raíz que ejecute ambas suites.
