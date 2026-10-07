# Flask on Docker

![Build](https://github.com/kseniia1803/flask-on-docker/actions/workflows/build.yml/badge.svg)

## Overview

This project is a containerized Flask web application backed by a PostgreSQL database, with separate development and production configurations orchestrated by Docker Compose. In development, the app runs on Flask's built-in server with live code reloading. In production, requests are served by Gunicorn, a production-grade WSGI server, behind an Nginx reverse proxy that also serves static files and user-uploaded media directly from shared Docker volumes. The production image uses a multi-stage build to stay small, lints the code with flake8 during the build, and runs as a non-root user for security. Together, these services form a simplified version of the stack used by large-scale web applications like Instagram.

![Demo: uploading and viewing an image](demo.gif)

## Build Instructions

### Prerequisites

- [Docker](https://docs.docker.com/get-docker/) with Docker Compose v2 (`docker compose`)

Clone the repository:

```bash
git clone https://github.com/kseniia1803/flask-on-docker.git
cd flask-on-docker
```

### Development

Build and start the web and database containers:

```bash
docker compose up -d --build
```

The app is available at http://localhost:1144. The database tables are created automatically on startup. To add a sample user to the database:

```bash
docker compose exec web python manage.py seed_db
```

To stop the services and remove the database volume:

```bash
docker compose down -v
```

### Production

The production database credentials are kept out of version control. Create a `.env.prod.db` file in the project root:

```bash
cat > .env.prod.db << 'END'
POSTGRES_USER=hello_flask
POSTGRES_PASSWORD=hello_flask
POSTGRES_DB=hello_flask_prod
END
```

If you choose a different password, update the `DATABASE_URL` line in `.env.prod` to match.

Build and start the web, database, and Nginx containers, then create the database tables:

```bash
docker compose -f docker-compose.prod.yml up -d --build
docker compose -f docker-compose.prod.yml exec web python manage.py create_db
```

The app is available at http://localhost:1144, served through Nginx.

To stop the services and remove all volumes:

```bash
docker compose -f docker-compose.prod.yml down -v
```

### Usage

| URL | Description |
| --- | --- |
| `/` | Returns a JSON "hello world" response |
| `/static/hello.txt` | A static file (served by Nginx in production) |
| `/upload` | Form for uploading a file |
| `/media/<filename>` | Displays an uploaded file |

To try it out, go to `/upload`, choose an image, click **Upload**, then visit `/media/<your-file-name>` to view it.

If port 1144 is already in use on your machine, change the left-hand port number under `ports:` in the compose files (for example `8080:5000` or `8080:80`).
