# ---- builder ----
ARG RUST_VERSION=1.90.0
FROM rust:${RUST_VERSION}-trixie AS builder

RUN apt-get update \
 && apt-get install -y --no-install-recommends \
      build-essential ca-certificates clang cmake curl g++ git libclang-dev make pkg-config \
 && apt-get install -y --no-install-recommends \
      libbz2-dev liblz4-dev libsnappy-dev libzstd-dev zlib1g-dev \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /src
# Select repo and ref at build time:
#   REPO=https://github.com/blockstream/electrs.git  REF=new-index
#   REPO=https://github.com/romanz/electrs.git       REF=master
ARG REPO=https://github.com/blockstream/electrs.git
#ARG REF=new-index
#RUN git clone --filter=blob:none --depth 1 --branch "${REF}" "${REPO}" .
ARG REF=f1823d82b03dbd1dfef53f7b3611128dd2f4c1d2
RUN git clone --filter=blob:none "${REPO}" . \
 && git fetch origin "${REF}" \
 && git checkout "${REF}"

# Features:
#   electrum-discovery       -> discovery support
#   liquid                   -> Elements/Liquid support
# Example: --build-arg FEATURES="electrum-discovery,liquid"
ARG FEATURES=electrum-discovery
RUN cargo build --locked --release --bin electrs --features "${FEATURES}"

# ---- runtime ----
FROM debian:trixie-slim

RUN apt-get update \
 && apt-get install -y --no-install-recommends ca-certificates libstdc++6 curl \
 && rm -rf /var/lib/apt/lists/*

# Non-root user and data dir
RUN useradd -r -d /var/lib/electrs -s /usr/sbin/nologin electrs \
 && mkdir -p /data && chown -R electrs:electrs /data

COPY --from=builder /src/target/release/electrs /usr/local/bin/electrs

USER electrs
VOLUME ["/data"]

# Electrum TCP and HTTP API
EXPOSE 50001 3000

# Metadata
ARG DESCRIPTION="Electrs backend API optimized for public usage by Blockstream for the Esplora block explorer based on the re-implementation of romanz/electrs"
LABEL org.opencontainers.image.description=$DESCRIPTION

ENTRYPOINT ["/usr/local/bin/electrs"]
CMD ["--db-dir","/data","--electrum-rpc-addr","0.0.0.0:50001","--http-addr","0.0.0.0:3000"]