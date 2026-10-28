# ---------- Stage 1: build wheels ----------
    FROM python:3.12-slim-bookworm AS builder
    ENV PIP_NO_CACHE_DIR=1 PIP_DISABLE_PIP_VERSION_CHECK=1
    RUN apt-get update && apt-get install -y --no-install-recommends build-essential libpq-dev \
        && rm -rf /var/lib/apt/lists/*
    WORKDIR /wheels
    COPY requirements.txt .
    RUN pip wheel --wheel-dir /wheels -r requirements.txt
    
    # ---------- Stage 2: runtime ----------
    FROM python:3.12-slim-bookworm
    ENV PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1 PIP_NO_CACHE_DIR=1
    
    # GeoDjango runtime libs (GDAL/GEOS/PROJ) + libpq + pg_isready for the wait loop
    RUN apt-get update && apt-get install -y --no-install-recommends \
            libpq5 postgresql-client gdal-bin libgdal-dev libgeos-dev libproj-dev \
        && rm -rf /var/lib/apt/lists/*
    
    # Non-root user
    RUN groupadd --system --gid 1000 app && useradd --system --uid 1000 --gid app --create-home app
    WORKDIR /app
    
    COPY --from=builder /wheels /wheels
    COPY requirements.txt .
    RUN pip install --no-index --find-links=/wheels -r requirements.txt && rm -rf /wheels
    
    COPY --chown=app:app . .
    RUN mkdir -p /app/staticfiles /app/media && chown -R app:app /app/staticfiles /app/media \
        && chmod +x /app/entrypoint.sh
    
    USER app
    EXPOSE 8000
    ENTRYPOINT ["/app/entrypoint.sh"]
    CMD ["gunicorn", "config.wsgi:application", "--bind", "0.0.0.0:8000", "--workers", "3", "--access-logfile", "-"]