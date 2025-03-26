VERSION 0.8

ARG --global image_namespace=registry.makerforce.io/k3s-coreos
ARG --global image_tag=latest
ARG --global base_image=docker.io/library/alpine:3.21.3


#coreos-assembler-pull:
#	FROM quay.io/coreos-assembler/coreos-assembler:latest
#	SAVE IMAGE --push $image_namespace/cache/coreos-assembler:$image_tag

coreos-assembler-source:
	FROM $base_image

	RUN apk add --no-cache git

	GIT CLONE \
		--branch 846c0b0ad562eccd66ce6fe239e28234ee7e3264 \
		https://github.com/coreos/coreos-assembler.git \
		src

	SAVE ARTIFACT src/* /
	SAVE IMAGE --push $image_namespace/cache/coreos-assembler-source:$image_tag

coreos-assembler:
	BUILD +coreos-assembler-source

	FROM DOCKERFILE +coreos-assembler-source/

	SAVE IMAGE --push $image_namespace/cache/coreos-assembler:$image_tag


setup:
	BUILD +coreos-assembler

	FROM +coreos-assembler

	ARG source=/src
	ARG custom_overlay=overlay.d/99custom

	COPY . $source

	# Download and install k3s
	RUN mkdir -p $source/$custom_overlay/usr/bin \
		&& cd $source/$custom_overlay \
		&& curl -Lo usr/bin/k3s https://github.com/k3s-io/k3s/releases/download/v1.32.2%2Bk3s1/k3s \
		&& chmod 755 usr/bin/k3s

	#ARG COSA_NO_KVM=1
	ARG COSA_SKIP_OVERLAY=1
	RUN --privileged \
		cosa init --transient $source \
		&& cosa fetch

	SAVE IMAGE --push $image_namespace/cache/setup:$image_tag


build:
	BUILD +setup

	FROM +setup

	#ARG COSA_NO_KVM=1
	ARG COSA_SKIP_OVERLAY=1
	RUN --privileged \
		cosa fetch \
		&& cosa build container \
		&& cosa osbuild qemu metal metal4k \
		&& cosa buildextend-live

	SAVE ARTIFACT builds/* AS LOCAL artifacts

test:
	FROM docker.io/library/alpine:3.21.3

	RUN mkdir builds \
		&& touch builds/test
	SAVE ARTIFACT builds/* AS LOCAL artifacts/
