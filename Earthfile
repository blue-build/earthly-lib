VERSION 0.8

PROJECT blue-build/earthly-lib

all:
	BUILD --platform=linux/amd64 --platform=linux/arm64 ./cargo+build
	BUILD ./rust+build-images
