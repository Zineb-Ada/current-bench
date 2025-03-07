FROM ocaml/opam:debian-11-ocaml-4.13 AS build

RUN sudo apt-get update && \
    sudo apt-get install -qq -yy \
    pkg-config \
    m4 \
    libssl-dev \
    libsqlite3-dev \
    libpq-dev \
    libgmp-dev \
    libffi-dev \
    libev-dev \
    libcapnp-dev \
    graphviz \
    capnproto

RUN opam remote add origin 'https://opam.ocaml.org' --all-switches && opam remote remove default && opam update

WORKDIR /mnt/project

# Build dependencies.
COPY --chown=opam:opam pipeline/pipeline.opam pipeline.opam
RUN opam install -y --deps-only .
COPY --chown=opam pipeline/. .
RUN sudo chown opam .

# Build the project.
RUN opam exec -- dune build --profile=release bin/main.exe

FROM debian:stable-slim

ARG TARGET_ARCH=amd64

RUN apt-get update \
    && apt-get install --no-install-recommends --assume-yes \
        netbase \
        ca-certificates \
        apt-transport-https \
        curl \
        netcat \
        postgresql-client \
        gnupg \
        lsb-release \
        git \
        libpq-dev \
        libsqlite3-dev \
        libev-dev \
    && curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg \
    && echo "deb [arch=${TARGET_ARCH} signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" \
        > /etc/apt/sources.list.d/docker.list \
    && apt-get update \
    && apt-get install docker-ce-cli graphviz --assume-yes \
    && rm -r /var/lib/apt/lists /var/cache/apt

WORKDIR /app
ENV PATH="/app/bin:${PATH}"

COPY ./pipeline/aslr_seccomp.json /app/aslr_seccomp.json
COPY ./pipeline/db/migrations /app/db/migrations
COPY --from=build /home/opam/.opam/4.13/bin/omigrate /app/bin/omigrate
COPY --from=build /home/opam/.opam/4.13/bin/ocluster-admin /app/bin/ocluster-admin
COPY --from=build /mnt/project/_build/default/bin/main.exe /app/bin/current-bench-pipeline
COPY ./pipeline/entrypoint.sh /app/entrypoint.sh
COPY ./environments/production.conf /app/production.conf
