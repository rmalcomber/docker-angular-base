# Angular Base
Base docker image for serving an angular app, using NGINX, with 404 handling on client side routing.

This purpose of this image is to wrap an angular application inside a container with an NGINX webserver. Current images that existed didn't support handling the 404's that arise from client side routing without switching to a hashing strategy. This handles that with no need for hashing strategy within angular routing. 

Internal port is port 80. 

# Tags

- `latest` is the current tag and what should be used.
- `beta` will be the latest in development, although that doesn't exist, so don't use it.

# Tested
Currently tested on Angular 9+. 

# Building
- Clone the ropo

`docker build -t {TAG NAME} .`

# Todo
- Create tests
- Test on previous versions of angular

# Using the image

## Prebuilt Angular app

`Dockerfile `
``` yaml
FROM rmalcomber/angular-base:latest
COPY . /usr/share/nginx/html

```

Command: `docker build -t {image name}:[tag] {./path/to/angular/dist/output}`


## Building the Angular App (*The docker way*)


`Dockerfile`

``` yaml
# Stage 1: Build node_modules for caching purposes
FROM node:14.3.0-alpine3.10 as npmpackagecache
WORKDIR /app
COPY package*.json /app/
RUN npm install

# Stage 2: Build angular app with node_modules from cache
FROM node:14.3.0-alpine3.10 as builder
WORKDIR /app
COPY . /app
COPY --from=npmpackagecache /app/node_modules/ /app/node_modules/
ARG configuration=production
ARG project

RUN npm run build -- --outputPath=./dist/out --configuration $configuration $project

# Stage 3: Build nginx container with application
FROM rmalcomber/angular-base:latest as final
COPY --from=builder /app/dist/out /usr/share/nginx/html
```

`$ docker build -t {image name}:[tag] {./path/to/angular/root}`

Configuration type can be passed through with the `build-arg` command.

`$ docker build -t {image name}:[tag] --build-arg configuration=production {./path/to/angular/root}`

If working with workspaces and multiple projects, the project can be passed through with `build-arg`'s too. 

`$ docker build -t {image name}:[tag] --build-arg configuration=production --build-arg project=my-project-name {./path/to/angular/root}`
