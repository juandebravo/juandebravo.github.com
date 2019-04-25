---
layout: post
title: "How to skip a subfolder while mounting a volume in Docker"
categories: [docker]
---
When working in a containers based environment, it's very frequent to build Docker images that
are agnostic to the environment. That way, they can be deployed in any environment, from local
development to production, which gives the great advantage of preventing any change in a Docker
image taht was certified internally in a testing/staging environment for deploying it into
production.

However, this may create an issue while developing, where you want to test your changes as soon
as possible, hence it's convenient to skip creating a new Docker image every time a change in the
source code is detected.

A common pattern is to run locally a Docker container mounting the folder where your source code is
located into the folder where it's included while generating the Docker image (assuming there's no
compilation / packaging / etc for going straight into the point :) ).

For example, let's assume a *Node.js application* that keeps the source code in the root folder of
the repository in the host machine and that copies it into the
*/usr/src/app* folder of the Docker image. Mounting the source code folder can be accomplished
with the following command:

```
docker run -v $(PWD):/usr/src/app my-app:latest
```

While this is very straight forward, it may create a big issue, as you **don't want, at all,
to mount your node_modules subfolder into the Docker container**. Some dependencies
might require native code/compilation, and if your host machine does not match the one used in the
Docker image, you'd get into troubles.

To prevent this, a special volume option pointing to the *node_modules subfolder* can be included,
and this folder won't be overwritten by the host information:

```
docker run \
  -v $(PWD):/usr/src/app \
  -v /usr/src/app/node_modules \
  my-app:latest
```
