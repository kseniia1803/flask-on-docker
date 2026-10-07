# Flask on Docker

![Build](https://github.com/kseniia1803/flask-on-docker/actions/workflows/build.yml/badge.svg)

## Overview

This repository contains a Flask web application that runs entirely in Docker, using PostgreSQL for the database, Gunicorn as the application server, and Nginx as a reverse proxy. It has two configurations: a development setup that uses Flask's built-in server with live reloading, and a production setup where Nginx forwards requests to Gunicorn and serves static files and user uploads directly. Users can upload an image through a web form and then view it in the browser. A GitHub Actions workflow builds the development containers on every push to confirm the project builds successfully.

![Uploading and viewing an image](demo.gif)

## Build Instructions

You need [Docker](https://docs.docker.com/get-docker/) with Docker Compose installed.

```bash
git clone https://github.com/kseniia1803/flask-on-docker.git
cd flask-on-docker
```

### Development

```bash
docker compose up -d --build
```

The app runs at http://localhost:1144, and the database tables are created automatically. To add a sample user:

```bash
docker compose exec web python manage.py seed_db
```

Stop it with `docker compose down -v`.

### Production

The database credentials file is excluded from version control, so create `.env.prod.db` in the project root first:

```
POSTGRES_USER=hello_flask
POSTGRES_PASSWORD=hello_flask
POSTGRES_DB=hello_flask_prod
```

The password must match the one in the `DATABASE_URL` line of `.env.prod`. Then build the services and create the database tables:

```bash
docker compose -f docker-compose.prod.yml up -d --build
docker compose -f docker-compose.prod.yml exec web python manage.py create_db
```

The app runs at http://localhost:1144, served through Nginx. Stop it with `docker compose -f docker-compose.prod.yml down -v`.

### Using the app

| URL | What it does |
| --- | --- |
| `/` | Returns `{"hello": "world"}` |
| `/static/hello.txt` | Serves a static file |
| `/upload` | Upload form: choose a file and click **Upload** |
| `/media/<filename>` | Displays an uploaded file |

If the app runs on a remote server, forward the port to your computer with `ssh -L 1144:localhost:1144 user@server` and open the URLs in your local browser. If port 1144 is taken, change the left-hand number under `ports:` in the compose files.

## Changes from the Tutorial

This project follows the [TestDriven.io tutorial](https://testdriven.io/blog/dockerizing-flask-with-postgres-gunicorn-and-nginx/), with a few fixes needed for current software versions:

- Switched the base image from `python:3.11.3-slim-buster` to `python:3.11-slim-bookworm`, since Debian Buster's package repositories are no longer available.
- Installed `netcat-openbsd` instead of `netcat`, which no longer exists as an installable package on Bookworm.
- Changed the database URL to `postgresql+psycopg2://`, because newer SQLAlchemy versions default to a different Postgres driver that isn't installed.
- Increased Nginx's upload limit to 20 MB so image uploads larger than 1 MB succeed.
- Changed the port to 1144 to avoid conflicts on a shared server.
