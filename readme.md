# Docker for JavaScript Developers

## Purpose

This repo contains notes and slides for a conference talk titled **Docker for JavaScript Developers**

## Demo App

The demo application for this talk can be found at [docker-music](https://github.com/shanebarringer/docker-music)

## Notes

The in-depth speaker notes can be found at [Docker-for-JS-devs.md](https://github.com/shanebarringer/talk-docker-for-js-devs/blob/master/Docker-for-JS-devs.md)

## Slides

Slides can be viewed by:

- cloning the repo

```shell
$ git clone https://github.com/shanebarringer/talk-docker-for-js-devs.git
```

- starting the slides container

```shell
$ docker run -it -p 8081:8080 -v $(pwd):/app/slides msoedov/hacker-slides
```

This command will spin up a docker container with slides hosted at [http://localhost:8081](http://localhost:8081)
