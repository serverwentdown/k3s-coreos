VERSION 0.8

ARG --global image_namespace=registry.makerforce.io/k3s-coreos
ARG --global image_tag=latest
ARG --global base_image=docker.io/library/alpine:3.21.3


coreos-assembler-pull:
	FROM quay.io/coreos-assembler/coreos-assembler:latest
	SAVE IMAGE --push $image_namespace/cache/coreos-assembler:$image_tag

#coreos-assembler-source:
#	FROM $base_image
#
#	RUN apk add --no-cache git
#
#	GIT CLONE \
#		--branch b31a7d3e558058f79232005c75fbc9f4511cf342 \
#		https://github.com/coreos/coreos-assembler.git \
#		src
#	SAVE ARTIFACT src/* /
#
#coreos-assembler:
#	FROM DOCKERFILE +coreos-assembler-source/
#	SAVE IMAGE --push $image_namespace/coreos-assembler:$image_tag


setup:
	FROM +coreos-assembler-pull

	COPY . /src
	RUN cosa init --transient /src
	#ARG COSA_NO_KVM=1
	ARG COSA_SKIP_OVERLAY=1
	RUN --privileged cosa fetch

	SAVE IMAGE --push $image_namespace/cache/setup:$image_tag


build:
	FROM +setup

	#ARG COSA_NO_KVM=1
	ARG COSA_SKIP_OVERLAY=1
	RUN --privileged cosa fetch
	RUN --privileged cosa build container
	RUN --privileged cosa osbuild qemu metal metal4k
	RUN --privileged cosa buildextend-live
	RUN rm \
		builds/builds.json \
		builds/latest

	SAVE ARTIFACT --symlink-no-follow builds/*
