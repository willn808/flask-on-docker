# Flask on Docker

![build](https://github.com/willn808/flask-on-docker/actions/workflows/build.yml/badge.svg)

## Overview

This repo runs a small Flask web app on a production-style stack modeled on Instagram's architecture, with every service in its own Docker container managed by Docker Compose. Requests come in through an Nginx reverse proxy. Nginx serves static files and user-uploaded images directly and forwards everything else to the Flask app, which runs under the Gunicorn application server and stores its data in a PostgreSQL database. The repo includes two configurations: a development setup that uses Flask's built-in server and reloads code changes instantly, and a production setup that uses Gunicorn, Nginx, a multi-stage image build with linting, and a non-root container user.

![demo](demo.gif)

## Build Instructions

You need [Docker](https://docs.docker.com/get-docker/) with the Compose plugin. The app is published on port `1142`; to use a different port, change the left-hand number of the `ports` entry in `docker-compose.yml` and `docker-compose.prod.yml`.

### Development

```bash
docker compose up -d --build
```

- `http://localhost:1142/` returns a JSON hello-world response
- `http://localhost:1142/upload` lets you upload an image
- `http://localhost:1142/media/<filename>` displays the uploaded image

Stop the services with `docker compose down -v`.

### Production

The production database credentials are kept out of version control. Create a file named `.env.prod.db` in the repo root:

```
POSTGRES_USER=hello_flask
POSTGRES_PASSWORD=hello_flask
POSTGRES_DB=hello_flask_prod
```

If you choose a different username or password, update `DATABASE_URL` in `.env.prod` to match. Then build, start, and create the database tables:

```bash
docker compose -f docker-compose.prod.yml up -d --build
docker compose -f docker-compose.prod.yml exec web python manage.py create_db
```

The same URLs as above now work through Nginx. Stop the services with `docker compose -f docker-compose.prod.yml down -v`.
