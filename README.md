# Flask on Docker

![build](https://github.com/willn808/flask-on-docker/actions/workflows/build.yml/badge.svg)

## Overview

This is a small Flask web app running on the same kind of stack Instagram started with, where each service gets its own Docker container. Nginx sits in front and takes every request. It serves static files and uploaded images itself and passes everything else to Gunicorn, which runs the Flask app, and the app keeps its data in Postgres. There are two setups: a development one that uses Flask's built-in server and picks up code changes without a rebuild, and a production one that adds Gunicorn, Nginx, a two-stage image build that lints the code first, and a non-root user inside the container.


![demo](demo.gif)

## Build Instructions

You need Docker with the Compose plugin. The site is published on port 1142. If that port is taken on your machine, change the left number in the `ports` line of `docker-compose.yml` and `docker-compose.prod.yml`.


### Development

```bash
docker compose up -d --build
```

Once it's up, `http://localhost:1142/` returns `{"hello": "world"}`. Go to `http://localhost:1142/upload` to upload an image, then open it at `http://localhost:1142/media/<filename>`. Uploaded filenames get cleaned up, so spaces turn into underscores.

To stop everything and wipe the database:

```bash
docker compose down -v
```
### Production

The production database password lives in `.env.prod.db`, which is left out of this repo on purpose. Create it in the repo root:

```
POSTGRES_USER=hello_flask
POSTGRES_PASSWORD=hello_flask
POSTGRES_DB=hello_flask_prod
```

If you pick a different username or password, change `DATABASE_URL` in `.env.prod` to match, or the app can't log into its own database.

Then build, start, and create the tables. Production doesn't create tables on startup, since that would wipe the data on every restart, so this step is manual and only needed once:

```bash
docker compose -f docker-compose.prod.yml up -d --build
docker compose -f docker-compose.prod.yml exec web python manage.py create_db
```

The same URLs work, now going through Nginx. To stop:

```bash
docker compose -f docker-compose.prod.yml down -v
```

