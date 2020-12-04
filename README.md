[![Build Status](https://jenkins.ricardodlc.com/buildStatus/icon?style=flat-square&job=NodeJs+Image)](https://jenkins.ricardodlc.com/job/NodeJs%20Image/)

Minimal NodeJS Docker Image
===========================

A minimal NodeJS image with Time Zone configurable.

---

This image is build on [Alpine Linux](https://hub.docker.com/_/alpine) 3.11 image. It comes with the necessary stuff to run/develop NodeJS applications and no unnecessary programs. It comes with bundled:

- NodeJS (See the current builded version, or check [Docker Hub](https://hub.docker.com/repository/docker/ricardodlc/minimal-nodejs-image) for other versions)
- NPM
- Yarn

## Build Your Own Image

The build process is very simple, just execute the following command:

```console
$ docker build -t image-name . --squash
```

> **Note:** the `--squash` argument its available only in **API 1.25+**. It squash newly built layers into a single new layer. This helps to reduce the final image size. See the [documentation](https://docs.docker.com/engine/reference/commandline/image_build/) for more info.

### Configure A Time Zone

Since this image comes with `America/Cancun` time zone by default, you can customize by your prefered time zone by using the `TZ` argument like this:

```console
$ docker build -t image-name --build-arg TZ=America/Cancun . --squash
```

Just replace the `TZ` value with your desired time zone.
