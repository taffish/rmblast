ARG RMBLAST_VERSION=2.17.1
ARG RMBLAST_ARCHIVE=rmblast-2.17.1+-x64-linux.tar.gz
ARG RMBLAST_SHA256=9ddf0ffdc01f3dd7bf62bbcb01516757b1e732e54674e81d93a7c143157aa527
ARG DEBIAN_MIRROR=http://deb.debian.org/debian
ARG DEBIAN_SECURITY_MIRROR=http://deb.debian.org/debian-security

FROM debian:12-slim AS fetcher

ARG RMBLAST_VERSION
ARG RMBLAST_ARCHIVE
ARG RMBLAST_SHA256
ARG DEBIAN_MIRROR
ARG DEBIAN_SECURITY_MIRROR
ARG TARGETARCH=amd64

ENV DEBIAN_FRONTEND=noninteractive

RUN set -eux; \
    case "${TARGETARCH}" in \
      amd64|"") ;; \
      *) echo "RMBlast official Linux binary is x86_64 only; unsupported TARGETARCH=${TARGETARCH}" >&2; exit 1 ;; \
    esac; \
    sed -i \
      -e "s|http://deb.debian.org/debian-security|${DEBIAN_SECURITY_MIRROR}|g" \
      -e "s|http://deb.debian.org/debian|${DEBIAN_MIRROR}|g" \
      /etc/apt/sources.list.d/debian.sources; \
    for attempt in 1 2 3 4 5 6; do \
      if apt-get -o Acquire::Retries=3 update \
       && apt-get -o Acquire::Retries=3 install -y --no-install-recommends \
            ca-certificates \
            curl \
            tar \
            gzip; then \
        break; \
      fi; \
      if [ "$attempt" = "6" ]; then exit 1; fi; \
      apt-get clean; \
      rm -rf /var/lib/apt/lists/*; \
      sleep 5; \
    done; \
    rm -rf /var/lib/apt/lists/*

WORKDIR /tmp/build

RUN set -eux; \
    url="https://www.repeatmasker.org/rmblast/${RMBLAST_ARCHIVE}"; \
    for attempt in 1 2 3 4 5 6; do \
      if curl \
          --fail \
          --location \
          --show-error \
          --connect-timeout 30 \
          --retry 3 \
          --retry-delay 5 \
          --retry-all-errors \
          -o "${RMBLAST_ARCHIVE}" \
          "${url}"; then \
        break; \
      fi; \
      if [ "$attempt" = "6" ]; then exit 1; fi; \
      sleep 10; \
    done; \
    echo "${RMBLAST_SHA256}  ${RMBLAST_ARCHIVE}" | sha256sum -c -; \
    mkdir -p /opt/rmblast; \
    tar -xzf "${RMBLAST_ARCHIVE}" -C /opt/rmblast --strip-components=1; \
    test -x /opt/rmblast/bin/rmblastn; \
    test -r /opt/rmblast/README.rmblast; \
    test -r /opt/rmblast/LICENSE.rmblast

FROM debian:12-slim

LABEL maintainer="taffish@163.com"

ARG RMBLAST_VERSION
ARG DEBIAN_MIRROR
ARG DEBIAN_SECURITY_MIRROR

ENV DEBIAN_FRONTEND=noninteractive

RUN set -eux; \
    sed -i \
      -e "s|http://deb.debian.org/debian-security|${DEBIAN_SECURITY_MIRROR}|g" \
      -e "s|http://deb.debian.org/debian|${DEBIAN_MIRROR}|g" \
      /etc/apt/sources.list.d/debian.sources; \
    for attempt in 1 2 3 4 5 6; do \
      if apt-get -o Acquire::Retries=3 update \
       && apt-get -o Acquire::Retries=3 install -y --no-install-recommends \
            ca-certificates \
            curl \
            gzip \
            libbz2-1.0 \
            libgomp1 \
            libsqlite3-0 \
            libstdc++6 \
            libzstd1 \
            perl \
            perl-doc \
            tar \
            zlib1g; then \
        break; \
      fi; \
      if [ "$attempt" = "6" ]; then exit 1; fi; \
      apt-get clean; \
      rm -rf /var/lib/apt/lists/*; \
      sleep 5; \
    done; \
    rm -rf /var/lib/apt/lists/*

COPY --from=fetcher /opt/rmblast /opt/rmblast

ENV PATH=/opt/rmblast/bin:${PATH}
ENV BLAST_USAGE_REPORT=false

RUN set -eux; \
    rmblastn -version 2>&1 | grep -F "rmblastn: ${RMBLAST_VERSION}+" >/dev/null; \
    blastn -version 2>&1 | grep -F "blastn: ${RMBLAST_VERSION}+" >/dev/null; \
    rmblastn -help 2>&1 | grep -F "Nucleotide-Nucleotide BLAST with RepeatMasker Extensions ${RMBLAST_VERSION}+" >/dev/null; \
    rmblastn -help 2>&1 | grep -F -- "-complexity_adjust" >/dev/null; \
    rmblastn -help 2>&1 | grep -F -- "-mask_level" >/dev/null; \
    ldd "$(command -v rmblastn)" | grep -F "libsqlite3" >/dev/null; \
    ldd "$(command -v rmblastn)" | grep -F "libgomp" >/dev/null; \
    ! ldd "$(command -v rmblastn)" | grep -F "not found" >/dev/null; \
    makeblastdb -help 2>&1 | grep -F "USAGE" >/dev/null; \
    blastdbcmd -help 2>&1 | grep -F "USAGE" >/dev/null; \
    legacy_blast.pl --help 2>&1 | grep -F "legacy_blast.pl" >/dev/null; \
    update_blastdb.pl --help 2>&1 | grep -F "update_blastdb.pl" >/dev/null; \
    mkdir -p /tmp/rmblast-check; \
    cd /tmp/rmblast-check; \
    printf '>s\nACGTTGCATGTCAGTACGATCGTACGATGCTAGCTAGGATCCGATCGTACGTTGCATGTCAGTACGATCGTACGATGCTAGCTAGGATCCGATCGT\n' > db.fa; \
    makeblastdb -in db.fa -dbtype nucl -out db >/dev/null; \
    printf '>q\nACGTTGCATGTCAGTACGATCGTACGATGCTAGCTAGGATCCGATCGT\n' > q.fa; \
    rmblastn -query q.fa -db db -task megablast -dust no -outfmt "6 qseqid sseqid pident length" \
      | awk '$1=="q" && $2=="s" && $3=="100.000" && $4>=48 {found=1} END {exit !found}'; \
    rm -rf /tmp/rmblast-check

WORKDIR /root

ENV TAFFISH_ENV=TAFFISH
ENV TAFFISH_NAME=rmblast
