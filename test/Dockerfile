# syntax=docker/dockerfile:1

# Adapted from:
# - https://hub.docker.com/r/docker/whalesay
# - https://github.com/docker/whalesay

ARG DEBIAN_VERSION
FROM debian:$DEBIAN_VERSION

# Reduces output from `apt-get`
ENV DEBIAN_FRONTEND="noninteractive"

# Configure `apt-get` to keep downloaded packages. Needed for using `--mount=type=cache` with `apt-get`
# https://docs.docker.com/engine/reference/python-deps/#example-cache-apt-packages
RUN rm -f /etc/apt/apt.conf.d/docker-clean && \
    echo 'Binary::apt::APT::Keep-Downloaded-Packages "true";' | tee -a /etc/apt/apt.conf.d/keep-cache

RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get -qq update && \
    apt-get -qq install cowsay fortune && \
    mv /usr/share/cowsay/cows/default.cow /usr/share/cowsay/cows/cow.cow

# Add "cowsay" location onto the path
ENV PATH=$PATH:/usr/games

COPY docker.cow /usr/share/cowsay/cows/
RUN ln -sf /usr/share/cowsay/cows/docker.cow /usr/share/cowsay/cows/default.cow

CMD ["/bin/sh", "-c", "fortune | cowsay"]
