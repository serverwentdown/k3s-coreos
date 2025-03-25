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
	FROM DOCKERFILE +coreos-assembler-source/

	SAVE IMAGE --push $image_namespace/cache/coreos-assembler:$image_tag


setup:
	BUILD +coreos-assembler

	FROM +coreos-assembler

	COPY fedora-coreos-config /src/fedora-coreos-config
	COPY *.yaml *.repo /src

	#ARG COSA_NO_KVM=1
	ARG COSA_SKIP_OVERLAY=1
	RUN --privileged \
		cosa init --transient /src \
		&& cosa fetch

	SAVE IMAGE --push $image_namespace/cache/setup:$image_tag


build:
	BUILD +setup

	FROM +setup

	COPY . /src

	# Download and install k3s
	RUN cd /src/overlay.d/99custom \
		&& curl -Lo usr/bin/k3s https://github.com/k3s-io/k3s/releases/download/v1.32.2%2Bk3s1/k3s \
		&& chmod 755 usr/bin/k3s \
		&& chown root:root usr/bin/k3s

	#ARG COSA_NO_KVM=1
	ARG COSA_SKIP_OVERLAY=1
	RUN --privileged \
		cosa fetch
	RUN --privileged \
		cosa build container
	RUN --privileged \
		cosa osbuild qemu
	RUN --privileged \
		cosa buildextend-live
	#	&& rm -rf \
	#		cache \
	#		tmp \
	#		builds/builds.json \
	#		builds/latest
	# Other osbuild targets: metal metal4k

	SAVE ARTIFACT builds/*
