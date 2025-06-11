# (blocos amd64, arm64 e builder permanecem iguais...)
FROM --platform=linux/amd64 debian:bookworm as debian-amd64
RUN apt-get update
RUN apt-get install -y libssh2-1-dev libssl-dev zlib1g-dev

FROM --platform=linux/arm64 debian:bookworm as debian-arm64
RUN apt-get update
RUN apt-get install -y libssh2-1-dev libssl-dev zlib1g-dev

FROM --platform=$BUILDPLATFORM golang:1.20.5-bookworm as builder
# … comandos go, dependências e build do binário DevLake inalterados …

FROM python:3.9-slim-bookworm as base

RUN apt-get update && \
    apt-get install -y python3-dev python3-pip tar pkg-config curl libssh2-1 zlib1g libffi-dev default-libmysqlclient-dev libpq-dev tini git openssh-client corkscrew && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /usr/share/doc/*

EXPOSE 8080

RUN useradd -ms /bin/bash -d /app devlake -u 1010
RUN chown -R devlake:devlake /etc/ssl/certs
USER devlake
ENV PYTHONUNBUFFERED=1

WORKDIR /app

RUN mkdir logs
VOLUME /app/logs

# Setup Python requirements
COPY python/ /app/python/
RUN python3 -m pip install --no-cache --upgrade pip setuptools && \
    python3 -m pip install --no-cache -r python/requirements.txt

# Instala Poetry
RUN curl -sSL https://install.python-poetry.org | python3 -
ENV PATH="$PATH:/app/.local/bin"

# Build dos plugins Python
RUN /app/python/build.sh

# 👉 Instala azure-devops via poetry
RUN poetry config virtualenvs.create false && \
    poetry add azure-devops

FROM base as devlake-base
ARG DEBUG=

# Copia bibliotecas compiladas
ENV LD_LIBRARY_PATH=/app/libs
RUN mkdir -p /app/libs
COPY --from=builder /usr/local/deps/target/lib/*.so* /app/libs
COPY --from=builder /go/bin /usr/bin
RUN ldconfig -vn /app/libs

# Copia binários e recursos
COPY --from=builder /app/bin /app/bin
COPY --from=builder /app/resources /app/resources

ENV PATH="/app/bin:${PATH}"
ENV DEBUG="$DEBUG"

# Usa o tini para evitar processos órfãos
ENTRYPOINT ["/usr/bin/tini", "--"]
CMD ["lake"]
