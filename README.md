# Docker Challenge — Repositorio plantilla

Este repositorio contiene una aplicación Flask ya implementada y un evaluador para el taller. Tu tarea es crear `Dockerfile` y `docker-compose.yml`.

1. Lee `ENUNCIADO.md`.
2. Crea los dos archivos requeridos en la raíz.
3. No modifiques `app.py`, `requirements.txt` ni el evaluador.
4. Sube el proyecto a tu repositorio individual de GitHub.
5. Revisa la pestaña **Actions** para consultar las pruebas.

Estructura final: añade `Dockerfile` y `docker-compose.yml` a los archivos existentes.

No subas tokens, contraseñas reales ni otros secretos. El workflow es una ayuda de verificación; el docente también revisará el repositorio y tu comprensión.

## Mi solución

**Repositorio:** https://github.com/lizarazo2005/docker-challenge-plantilla

### Archivos creados
- `Dockerfile`: imagen `python:3.11-slim`, `WORKDIR /app`, instala `requirements.txt`, copia el código, expone el puerto 5000 e inicia con `python app.py`.
- `docker-compose.yml`: servicio `web` (construido desde el Dockerfile, puerto `5000:5000`) y servicio `db` (imagen oficial `postgres:16`), con un volumen nombrado `datos_postgres`.
- `.dockerignore`: ya venía en la plantilla; excluye `.git`, `__pycache__` y `*.pyc`.

### Evidencia: docker compose ps

```
NAME                               IMAGE         COMMAND                  SERVICE   STATUS              PORTS
docker-challenge-plantilla-db-1    postgres:16   "docker-entrypoint.s…"   db        Up About a minute   5432/tcp
docker-challenge-plantilla-web-1   (imagen web)  "python app.py"          web       Up About a minute   0.0.0.0:5000->5000/tcp
```

### Evidencia: docker compose config

```yaml
name: docker-challenge-plantilla
services:
  db:
    environment:
      POSTGRES_DB: challenge
      POSTGRES_PASSWORD: challengepass
      POSTGRES_USER: challenge
    image: postgres:16
    volumes:
      - type: volume
        source: datos_postgres
        target: /var/lib/postgresql/data
  web:
    environment:
      DATABASE_URL: postgresql://challenge:challengepass@db:5432/challenge
    build:
      context: .
      dockerfile: Dockerfile
    depends_on:
      db:
        condition: service_started
    ports:
      - target: 5000
        published: "5000"
        protocol: tcp
networks:
  default:
    name: docker-challenge-plantilla_default
volumes:
  datos_postgres:
    name: docker-challenge-plantilla_datos_postgres
```

### ¿Por qué DATABASE_URL usa db y no localhost?
Docker Compose crea una red interna donde cada servicio se encuentra por su nombre, y `db` es el nombre del contenedor de PostgreSQL en esa red. Si usara `localhost`, el contenedor `web` buscaría la base de datos dentro de sí mismo, donde no hay PostgreSQL, y la conexión fallaría.

### ¿Qué pasa con los datos al hacer docker compose down y volver a iniciar?
`docker compose down` borra los contenedores y la red, pero no el volumen nombrado `datos_postgres`, donde PostgreSQL guarda sus datos. Al hacer `docker compose up -d` el contenedor `db` vuelve a montar ese volumen y las tareas siguen ahí. Lo comprobé con tres tareas:

```
id title
-- -----
 1 mi primera tarea
 2 hacer en la mañana
 3 hacer en la noche
```

Los datos solo se perderían con `docker compose down -v`, que también borra el volumen.

### Evaluación automática
El workflow de GitHub Actions pasó 21/21 comprobaciones.