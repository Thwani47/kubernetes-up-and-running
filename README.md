# Kubernetes Up & Running
## Chapter 1: Creating and Running Containers
- Kubernetes is a platform for creating, deploying, and managing distrbiuted applications. 
 - Before we can consider how to build these distributed apps, we need to consider how to build the containers that contain these applications that make pieces of our distributed systems.
 - Containers solve the problem of dependency management and encapsulation. 
 - Container images bundle a program and all its dependencies into a single artifact under a root filesystem
 - Containers fall into 2 main categories:
    - System containers --> These run a fill OS inside the container
    - Application containers --> These differ from system containers as they commonly run a single program
- A Dockerfile can be used to automate the container image creation process
- It is crucial to remember that files that are removed by subsequent image layers in the system are still present in the image, they are just inaccessible
- Every time we change an image layer, it changes every layer that comes after it.
    - In a Dockerfile, it makes sense to copy the `package.json` file and run `npm install` before copying the application source code, because a developer is most likey to change the source code more frequently than the dependencies.
- Docker multi-stage builds can be used to create smaller images. 
    - With multi-stage builds, we do not package the application as part of the build process.
    - This is useful because the generated image does not contain the build tools required to build the image, as these tools might be large and slow down our deployments