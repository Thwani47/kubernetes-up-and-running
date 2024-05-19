- Kubernetes is a platform for creating, deploying, and managing distributed applications.
- Before we can consider how we can build a distributed system, we need to consider how build the application container images that contain these applications and make up the pieces of our distributed system
- Container images solve the problem of dependency management and encapsulation
- Container images bundle a program and all its dependencies into a single artifact under a root filesystem
- Container images are constructed with a series of filesystem layers, where each layer inherits and modifies the layers that come before it
- Containers fall into two main categories:
	- **System containers** --> System containers seek to mimic virtual machines and often run a full boot process. 
	- **Application containers** --> Application containers differ from system containers in that they commonly run a single program.
- A **Dockerfile** can be used to automate the creation of a Docker container image
####  Example 1: Dockerfile
Create these two files:
```json:package.json
{
	"name": "simple-node",
	"version": "1.0.0",
	"description": "A sample simple application for Kubernetes Up & Running",
	"main": "server.js",
	"scripts":{
		"start": "node server.js"
	},
	"author": ""
}
```

```js:server.js
var express = require('express');

var app = express();

app.get('/', (req, res) => {
	res.send('Hello World!');
});

app.listen(3000, () => {
	console.log('Server is running on http://localhost:3000');
});
```

Run this on your terminal:
```bash
npm install express --save
```
In order to package the app as a Docker image, we need to create two additional files: **.dockerignore** and **Dockerfile**
```bash:.dockerignore
node_modules
```
```Dockerfile:Dockerfile
# start from a Node.js 21 image
FROM node:21

# specify the directory inside the image in which all commands will run
WORKDIR /usr/src/app

# Copy the package files and install dependencies
COPY package*.json ./
RUN npm install
RUN npm install express

# COPY all of the application files into the image
COPY . .

# Specify the default command to run when starting the container
CMD ["npm", "start"]
```

To create the `simple-node` image, run
```bash
docker build -t simple-node .
```
To run the `simple-node` image, run the following command and open [localhost:3000
](http://localhost:3000)
```bash
docker run --rm -p 3000:3000 simple-node
```
At this point, our `simple-node` image lives in the local Docker registry and is only accessible from our machine

### Optimizing Image Sizes
- It is crucial to remember that files that are removed by subsequent layers in the system are still present in the images, they are just inaccessible
- Every time we change a layer, it changes every layer that comes after it
- In general, you want to order your layers from least likely to change to most likely to change
	- In the `simple-node` example, we copy the `package*.json` files and install dependencies before we copy the source code. A developer is most likely to change the source code more frequently than the dependencies
### Multistage Image Builds
- One of the most common ways to accidentally build large images is to do the actual program compilation as part of the construction of the application container image
- Compiling code as part of the image build feels natural, and it is the easiest way to build a container image. But the problem is that it leaves all the unnecessary development tools, which are usually quite large, lying around inside your image and slowing down your deployments.
- To resolve this, Docker introduced `multistage builds`
- With multistage builds, instead of building a single image, Docker can produce multiple images, with each image considered a stage. Artifacts can be copied from one stage to another. 
- Consider this example. We have a React app that uses a Go backend. A simple Dockerfile might look like this:
```Dockerfile:Dockerfile
FROM golang:1.21-alpine

# Install Node and NPM
RUN apk update &&& ap upgrade && apk add --no-cache git nodejs bash npm

# Get the Go dependencies
RUN go get -u github.com/jteeuwen/go-binddata/...
RUN go get -u github.com/tools/godep
RUN go get github.com/kubernetes-up-and-running/kuard

WORKDIR /go/src/github.com/kubernetes-up-and-running/kuard

# Copy all soruce files
COPY . .

RUN build/build.sh

CMD ["/go/bin/kuard"]
```
This image contains all of the Go development tools and the tools used to build the React app. All these aren't required in the final application. The image, adds up to over `500MB`

We can optimize this image using Docker multistage builds as follows
```Dockerfile:Dockerfile
# Stage 1: Build
FROM golang:1.21-alpine AS build

# Install Node and NPM
RUN apk update &&& ap upgrade && apk add --no-cache git nodejs bash npm

# Install the Go Dependencies
RUN go get -u github.com/jteeuwen/go-binddata/...
RUN go get -u github.com/tools/godep
RUN go get github.com/kubernetes-up-and-running/kuard

WORKDIR /go/src/github.com/kubernetes-up-and-running/kuard

# Copy all source files
COPY . .

# Do the build
RUN build/build.sh

# STAGE 2: Deployment
FROM alpine

USER nobody:nobody
COPY --from=build /go/bin/kuard /kuard
CMD ["/kuard"]
```
This Dockerfile now contains two images, the **build image**, which contains the Go compiler, the React toolchain, and the source code, an the **deployment image**, which simply contains the compiled binary. The final image is somewhere around `20MB`. 