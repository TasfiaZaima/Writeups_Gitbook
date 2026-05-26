# Understanding Docker Through Real QA Automation Workflows

When I first started learning Docker, most tutorials focused heavily on commands:

```bash
docker pull hello-world
docker run hello-world
```

I could follow along, but I didn’t fully understand what Docker was actually doing behind the scenes.

Things became much clearer once I started using Docker in automation testing workflows. Instead of treating Docker as a collection of commands to memorize, I began understanding it as a way to create consistent, reproducible environments.

That shift made everything easier to reason about.

***

## Understanding Images vs Containers

One of the most important Docker concepts is understanding the difference between images and containers.

***

### Docker Images

A Docker image is essentially a reusable blueprint.

It contains:

* application dependencies
* runtime environment
* required tools
* project code
* configuration needed to run the application

Images are commonly created using a Dockerfile.

Example:

```dockerfile
FROM node:20
WORKDIR /app
COPY . .
RUN npm install
CMD ["npm", "test"]
```

Then the image is built using:

```bash
docker build -t my-image .
```

The `.` matters because it tells Docker to use the current directory as the build context.

Once I understood that images are immutable templates, Docker’s workflow started making much more sense.

***

### Docker Containers

A container is a running instance created from an image.

Example:

```bash
docker run my-image
```

When this command runs, Docker:

* creates a container from the image
* starts the defined process
* runs the application
* exits when the process finishes

Containers are designed to be disposable and reproducible, which makes them especially valuable for automation testing and CI/CD workflows.

Instead of manually configuring environments repeatedly, Docker allows the same environment to be recreated consistently whenever needed.

That consistency is one of Docker’s biggest advantages in QA automation.

***

## Why Some Containers Exit Immediately

One thing that initially felt confusing was this:

```bash
docker run busybox
```

The container starts and exits almost instantly.

The reason is simple: containers remain alive only while their main process is running. If there’s no long-running process, the container exits.

Understanding this helped clarify that Docker is process-oriented, not VM-oriented.

***

## Interactive vs Detached Mode

Docker containers behave differently depending on how they are started.

***

### Interactive Mode

```bash
docker run -it busybox
```

This opens an interactive shell session inside the container.

The container remains active as long as the session is open.

Useful for:

* debugging
* exploring environments
* running commands manually

***

### Detached Mode and Long-Running Processes

One subtle but important Docker behavior is that detached mode alone does not keep containers alive.

For example:

```bash
docker run -d busybox sleep 3600
```

This works because sleep 3600 becomes the container’s active foreground process.

A common misconception is assuming this would also stay alive:

```bash
docker run -d busybox
```

In reality, the container exits almost immediately because there is no long-running process inside it.

That distinction helped reinforce an important mental model:

Containers do not stay alive because they are detached.\
They stay alive because a foreground process is still running.

Useful commands while working with containers:

```bash
docker ps
docker ps -a
docker logs <container-id>
docker stop <container-id>
```

***

## Using Docker for Automation Testing

Docker started becoming genuinely useful once I integrated it into browser automation testing.

Instead of relying on:

* local machine configuration
* manually installed dependencies
* inconsistent browser versions
* environment-specific setups

everything runs inside the same isolated environment.

For example:

```bash
docker run --rm playwright-tests
```

This allows the same Playwright or Selenium tests to run consistently across:

* local development machines
* CI/CD pipelines
* staging environments
* shared QA systems

That level of reproducibility significantly reduces environment-related issues.

***

## Why .dockerignore Matters

Another important lesson was understanding the role of .dockerignore.

Without it, Docker sends unnecessary files into the build context during image creation.

Typical examples include:

* node\_modules
* test reports
* temporary files
* Git metadata
* logs

Benefits of using .dockerignore properly:

* smaller Docker images
* faster build times
* cleaner environments
* reduced cache invalidation

For larger automation projects, this makes a noticeable difference.

***

## Build vs Run: The Workflow That Finally Clicked

The biggest mental model shift for me was understanding the separation between building and running.

***

### Rebuild Only When Something Changes

You rebuild the image when:

* application code changes
* dependencies change
* Dockerfile configuration changes

Example:

```bash
docker build -t my-image .
```

***

### Otherwise, Just Run the Existing Image

```bash
docker run --rm my-image
```

The --rm flag automatically removes the container after execution, preventing stopped containers from piling up over time.

This distinction between immutable images and disposable containers made Docker workflows feel much more predictable.

***

## Understanding Docker Disk Usage

Over time, Docker can consume a significant amount of disk space due to:

* unused images
* stopped containers
* build cache
* volumes

A useful command for monitoring usage is:

```bash
docker system df
```

***

## Cleaning Up Docker Resources

***

### Remove Stopped Containers

```bash
docker container prune
```

This safely removes stopped containers that are no longer needed.

***

### Remove Unused Images

```bash
docker image prune
```

This removes dangling images — typically intermediate or untagged images left behind after rebuilds.

Docker also provides:

```bash
docker image prune -a
```

However, this should be used carefully.

In Docker terminology, “unused” means any image that is not currently attached to an existing container. That means even large browser images or cached automation environments can be deleted if no container is actively using them.

For automation projects, removing those images may significantly slow down future builds because Docker will need to pull everything again.

***

### Remove Build Cache

```bash
docker builder prune -a
```

This removes cached build layers that Docker stores to speed up image creation.

Useful occasionally, but aggressive cache cleanup can also increase future build times.

***

### Full System Cleanup

```bash
docker system prune -a --volumes
```

This is effectively the “nuclear cleanup” option.

It removes:

* stopped containers
* unused images
* unused networks
* build cache
* unused volumes

The --volumes flag is especially important to understand because volumes often contain persistent data.

For example:

* local databases
* mock test data
* uploaded files
* logs
* cached browser profiles

If those volumes are not attached to a running container when this command executes, the data can be permanently deleted.

Because of that, I learned to use this command only when I’m absolutely certain no important local data needs to be preserved.

***

## A Windows-Specific Observation

On Windows, Docker Desktop commonly runs using WSL2 internally.

One thing I noticed was that disk space does not always immediately shrink after deleting Docker resources.

In some cases, restarting Docker or shutting down WSL helps reclaim space:

```bash
wsl --shutdown
```

***

## Why Docker Fits Well in QA Workflows

The more I worked with Docker, the more it made sense for automation testing and QA engineering.

Docker provides:

* consistent environments
* isolated execution
* reproducible test setups
* easier CI/CD integration
* reduced environment-related failures

Instead of hearing:

“It works on my machine.”

Docker helps create environments where tests behave consistently everywhere.

And in QA automation, consistency is critical.

By packaging Playwright or Selenium, runtime dependencies, browser binaries, and environment configuration into immutable images, Docker helps eliminate differences between local machines and CI environments.

That significantly reduces flaky tests caused by inconsistent setups.

***

## Final Thoughts

Initially, Docker felt intimidating because of the number of commands and concepts involved.

But once I understood:

* images vs containers
* build vs run workflows
* container lifecycle behavior
* environment isolation
* cleanup and reproducibility

Docker stopped feeling complicated and started feeling predictable.

The biggest takeaway for me was this:

Docker is less about memorizing commands and more about creating reliable, reproducible environments.

And for QA automation, that becomes incredibly valuable.
