FROM ubuntu:24.04 AS clone-stage 

WORKDIR /src

# Install dependencies
RUN apt-get update && apt-get install -y \
    git \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Clone odoo project
RUN git clone --depth 1 https://github.com/odoo/odoo.git

FROM python:3.11.11-bookworm AS dependency-stage

WORKDIR /app

# # Install dependencies
RUN apt-get update && apt-get install -y \ 
    libldap2-dev \
    libpq-dev \
    libsasl2-dev \
    build-essential \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Copy odoo from clone-stage
COPY --from=clone-stage /src/odoo /app/odoo

# Install python requirements
RUN python3 -m venv /opt/venv \
    && /opt/venv/bin/pip install --upgrade pip setuptools wheel

# Config environment var
ENV PATH="/opt/venv/bin/:$PATH"

RUN cd /app/odoo && pip install -Ur requirements.txt

# Copy entrypoint script and make it executable
COPY entrypoint.sh /app/
RUN chmod +x /app/entrypoint.sh

FROM python:3.11.11-bullseye AS prod-stage

WORKDIR /app

RUN apt-get update && apt-get install -y \
    postgresql-client \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

COPY --from=dependency-stage /app/odoo /app/odoo
COPY --from=dependency-stage /opt/venv /opt/venv

ENV PATH="/opt/venv/bin:$PATH"

COPY --from=dependency-stage /app/entrypoint.sh /app/

# Port
EXPOSE 8069

# ENTRYPOINT
ENTRYPOINT [ "/app/entrypoint.sh" ]