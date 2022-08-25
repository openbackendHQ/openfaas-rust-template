ARG OF_WATCHDOG_VERSION=0.9.6
ARG RUST_VERSION=1.62
ARG ALPINE_VERSION=3.11

FROM openfaas/of-watchdog:${OF_WATCHDOG_VERSION} as watchdog


# Build Stage
FROM rust:${RUST_VERSION} AS builder

RUN USER=root cargo new --bin function
WORKDIR /function

COPY ./source/Cargo.lock ./Cargo.lock
COPY ./source/Cargo.toml ./Cargo.toml
COPY ./source/src ./src

RUN cargo build --release
RUN cargo install --root /app --path .

# ##########################################

FROM alpine:${ALPINE_VERSION} as runner

WORKDIR /app

# Install packages and add non-root user
RUN apk --no-cache add curl ca-certificates \
    && addgroup -S app && adduser -S -g app app

ENV USER=app

# Copy of-watchdog binary
COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog
RUN chmod +x /usr/bin/fwatchdog

COPY --from=build /app/bin/function .

# Set up watchdog for HTTP mode
ENV fprocess="./function"
ENV mode="http"
ENV upstream_url="http://127.0.0.1:8080"

HEALTHCHECK --interval=3s CMD [ -e /tmp/.lock ] || exit 1

CMD ["fwatchdog"]