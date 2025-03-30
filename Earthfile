VERSION 0.8

ARG --global image_namespace=registry.makerforce.io/k3s-coreos
ARG --global image_tag=latest
ARG --global base_image=docker.io/library/alpine:3.21.3


coreos-assembler-source:
	FROM $base_image

	RUN apk add --no-cache git

	GIT CLONE \
		--branch 4c2afc4b72954427a3d2f71fc73c1ea762ec19c1 \
		https://github.com/coreos/coreos-assembler.git \
		src

	SAVE ARTIFACT src/* /
	SAVE IMAGE --push $image_namespace/cache/coreos-assembler-source:$image_tag

coreos-assembler:
	BUILD +coreos-assembler-source
	FROM DOCKERFILE +coreos-assembler-source/

	SAVE IMAGE --push $image_namespace/cache/coreos-assembler:$image_tag


setup:
	FROM $image_namespace/cache/coreos-assembler:$image_tag

	ARG source=/src
	ARG custom_overlay=overlay.d/99custom

	COPY . $source

	# Create overlay binaries directory
	RUN mkdir -p $source/$custom_overlay/usr/bin

	# Download and install k3s
	RUN cd $source/$custom_overlay \
		&& curl -Lo usr/bin/k3s https://github.com/k3s-io/k3s/releases/download/v1.32.2%2Bk3s1/k3s \
		&& chmod 755 usr/bin/k3s

	# Download and install dust
	RUN cd $source/$custom_overlay \
		&& curl -Lo dust.tar.gz https://github.com/bootandy/dust/releases/download/v1.1.2/dust-v1.1.2-x86_64-unknown-linux-gnu.tar.gz \
		&& tar -xz --strip-components 1 '*dust' --directory usr/bin \
		&& rm dust.tar.gz

	ARG COSA_SKIP_OVERLAY=1
	RUN --privileged \
		cosa init --transient $source \
		&& cosa fetch

	SAVE IMAGE --push $image_namespace/cache/setup:$image_tag


build:
	FROM +setup

	ARG COSA_SKIP_OVERLAY=1
	RUN --privileged \
		cosa fetch \
		&& cosa build container \
		&& cosa osbuild qemu metal metal4k \
		&& cosa buildextend-live
	RUN rm \
		builds/latest \
		builds/builds.json

	SAVE ARTIFACT builds/* AS LOCAL artifacts/
