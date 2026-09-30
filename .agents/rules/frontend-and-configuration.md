---
title: "Conservar la configuración segura y coordinada del entorno"
description: "Coordina el acceso local a la API con Vite y Compose, evita exponer secretos y trata las configuraciones actuales como desarrollo."
scope: project
globs:
  - "frontend/src/**/*.ts"
  - "frontend/src/**/*.tsx"
  - "frontend/vite.config.ts"
  - "frontend/.env*"
  - "frontend/Dockerfile"
  - "backend/app/main.py"
  - "backend/Dockerfile"
  - "docker-compose.yml"
alwaysApply: false
---

Al modificar frontend, configuración o despliegue:

1. Si cambias `/api`, el puerto o el nombre del servicio `backend`, actualiza y verifica conjuntamente `frontend/src/App.tsx`, `frontend/vite.config.ts` y `docker-compose.yml`. Conserva `VITE_API_BASE_URL` como override opcional y revisa `frontend/.env.example` si cambia su propósito.
2. Usa variables `VITE_*` únicamente para valores que puedan exponerse al navegador. No incluyas tokens, contraseñas ni credenciales de servicios en el entorno del frontend.
3. Antes de cambiar `CORSMiddleware` o exponer el backend en otro entorno, determina los orígenes y métodos requeridos y configura CORS explícitamente. No copies automáticamente los comodines actuales a otro contexto.
4. Trata los Dockerfiles y `docker-compose.yml` actuales como el flujo de desarrollo demostrado. No los presentes como configuración de producción sin crear, documentar y verificar una configuración apropiada.

La configuración actual está en `frontend/src/App.tsx`, `frontend/vite.config.ts`, `frontend/.env.example`, `backend/app/main.py`, `frontend/Dockerfile`, `backend/Dockerfile` y `docker-compose.yml`. Vite proxifica `/api` a `http://backend:8000`; las variables `VITE_*` son visibles al cliente; CORS permite comodines y credenciales; los contenedores inician servidores de desarrollo/debug y Compose monta el código local.
