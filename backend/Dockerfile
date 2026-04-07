FROM python:3.12-slim AS base

ARG NEC2C_TARBALL_URL=https://github.com/KJ7LNW/nec2c/archive/55be1e0e3fe5ee9dad4ce6050711450d19c562fd.tar.gz
ARG NEC2C_TARBALL_SHA256=35c9c8cae99d9b95c628e70047352beb458090d1d9682c8529bc7a5bcf33dcfd

RUN set -eux; \
    apt-get update; \
    if apt-get install -y --no-install-recommends nec2c; then \
        echo "Installed nec2c from apt"; \
    else \
        echo "nec2c package unavailable; building from pinned source"; \
        apt-get install -y --no-install-recommends ca-certificates curl gcc libc6-dev; \
        curl -fsSL "${NEC2C_TARBALL_URL}" -o /tmp/nec2c.tar.gz; \
        echo "${NEC2C_TARBALL_SHA256}  /tmp/nec2c.tar.gz" | sha256sum -c -; \
        mkdir -p /tmp/nec2c-src; \
        tar -xzf /tmp/nec2c.tar.gz -C /tmp/nec2c-src --strip-components=1; \
        cc -O2 -pipe \
          -o /usr/local/bin/nec2c \
          /tmp/nec2c-src/main.c \
          /tmp/nec2c-src/calculations.c \
          /tmp/nec2c-src/fields.c \
          /tmp/nec2c-src/geometry.c \
          /tmp/nec2c-src/ground.c \
          /tmp/nec2c-src/input.c \
          /tmp/nec2c-src/matrix.c \
          /tmp/nec2c-src/misc.c \
          /tmp/nec2c-src/network.c \
          /tmp/nec2c-src/radiation.c \
          /tmp/nec2c-src/shared.c \
          /tmp/nec2c-src/somnec.c \
          -lm; \
        rm -rf /tmp/nec2c.tar.gz /tmp/nec2c-src; \
        apt-get purge -y --auto-remove gcc libc6-dev curl; \
    fi; \
    which nec2c; \
    nec2c -h >/tmp/nec2c-help.txt 2>&1 || true; \
    test -s /tmp/nec2c-help.txt; \
    rm -f /tmp/nec2c-help.txt; \
    apt-get clean; \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY backend/pyproject.toml ./
RUN pip install --no-cache-dir \
    "fastapi>=0.115.0" \
    "uvicorn[standard]>=0.32.0" \
    "pydantic>=2.10.0" \
    "pydantic-settings>=2.6.0" \
    "redis>=5.2.0" \
    "httpx>=0.28.0" \
    "scipy>=1.14.0"

COPY VERSION ./VERSION
COPY backend/src/ ./src/

RUN useradd -m -r -s /bin/false appuser && \
    mkdir -p /tmp/nec_workdir && \
    chown -R appuser:appuser /app /tmp/nec_workdir
USER appuser

ENV NEC_WORKDIR=/tmp/nec_workdir
ENV PYTHONUNBUFFERED=1

EXPOSE 8000
CMD ["uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "2"]
