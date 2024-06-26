#
# NOTE: THIS DOCKERFILE IS GENERATED VIA "apply-templates.sh"
#
# PLEASE DO NOT EDIT IT DIRECTLY.
#

FROM --platform=$BUILDPLATFORM golang:1.21 AS build

# https://github.com/moby/buildkit/releases
ENV BUILDKIT_VERSION 0.13.2

COPY \
	containerd-arm64-v8.patch \
	git-no-submodules.patch \
	mips64le.patch \
	noclip.patch \
	/tmp/buildkit-patches/

WORKDIR /app

RUN set -eux; \
	git clone --branch "v$BUILDKIT_VERSION" --depth 1 --config advice.detachedHead=false 'https://github.com/moby/buildkit.git' .; \
	git apply /tmp/buildkit-patches/*.patch; \
	git diff --color

ENV CGO_ENABLED 0

ARG TARGETOS TARGETARCH TARGETVARIANT
ENV GOOS=$TARGETOS GOARCH=$TARGETARCH VARIANT=$TARGETVARIANT

RUN set -eux; \
	case "$GOARCH" in \
		arm) export GOARM="${VARIANT#v}" ;; \
		amd64) export GOAMD64="$VARIANT" ;; \
		*) [ -z "$VARIANT" ] ;; \
	esac; \
	go env | grep -E 'OS=|ARCH=|ARM=|AMD64='; \
# https://github.com/moby/buildkit/blob/v0.13.2/Dockerfile#L93
	pkg='github.com/moby/buildkit'; \
	ldflags=" \
		-d -w \
		-X '$pkg/version.Version=$BUILDKIT_VERSION-tianon' \
		-X '$pkg/version.Revision=v$BUILDKIT_VERSION+tianon-patches' \
		-X '$pkg/version.Package=$pkg' \
	"; \
	go build -o /buildkitd -trimpath -ldflags "$ldflags" ./cmd/buildkitd; \
	go build -o /buildctl -trimpath -ldflags "$ldflags" ./cmd/buildctl

FROM --platform=$TARGETPLATFORM infosiftr/moby

RUN set -eux; \
	apt-get update; \
	apt-get install -y --no-install-recommends git; \
	rm -rf /var/lib/apt/lists/*

COPY --from=build --link /buildkitd /buildctl /usr/local/bin/
COPY buildkitd-entrypoint.sh /usr/local/bin/

VOLUME /var/lib/buildkit

# https://github.com/docker/buildx/issues/484#issuecomment-749352728
ENV BUILDKIT_STEP_LOG_MAX_SIZE -1
ENV BUILDKIT_STEP_LOG_MAX_SPEED -1

ENTRYPOINT ["buildkitd-entrypoint.sh"]
CMD []
