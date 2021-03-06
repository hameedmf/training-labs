# Labs: Build

## Build a static binary

Clone the training repo:

```
cd
git clone https://github.com/boxboat/training-labs
cd training-labs/go-example
```

View the Dockerfile and hello.go. This is an extremely simple "hello world"
go image. Run a build:

```
docker image build -t my-hello:static .
```

Test this app out:

```
docker container run -it --rm my-hello:static
```

## Updating an Image

Try editing the hello.go file and rerunning the container to see the image is
unchanged. Rebuild the image with another tag "my-hello:v2", and run the old and
new versions to see how easy it is to change between running versions.

View the history of your old and new images with:

```
docker image history my-hello:static
docker image history my-hello:v2
```

Where do the layers diverge between the old and new images?

## Multi-stage Build

View the Dockerfile.multi-stage, compare it to the previous Dockerfile. Build
this image:

```
docker image build -f Dockerfile.multi-stage -t my-hello:multi-stage .
```

Verify the image still works:

```
docker container run -it --rm my-hello:multi-stage
```

What happens if you try to get a shell inside this container and why? Hint:
check the value of your base image.

```
docker container run -it --rm my-hello:multi-stage /bin/sh
```

Compare the history of this image with the static image you built before:

```
docker image history my-hello:static
docker image history my-hello:multi-stage
```

Compare the image sizes:

```
docker image ls my-hello
```

Which image is easier for developers to debug? Which image would be more secure
for production and faster to deploy in the cloud?

We can have the best of both, lets build just the first stage of the
multi-stage image:

```
docker image build -f Dockerfile.multi-stage --target build -t my-hello:build .
```

Try running the "my-hello:build" image to verify it works, compare the history
and image size, and see if you can get a shell into the container:

```
docker container run -it --rm my-hello:build
docker image history my-hello:build
docker image ls my-hello
docker container run -it --rm my-hello:build /bin/sh
```

Inside the container shell, try running your app, try running some debugging
commands and then exit:

```
app
ls -l
ls -l /go/bin/app
ldd /go/bin/app
env
ps -ef
ip a
exit
```

## Dockerignore file

When you did a list in the source of the container above, did you notice the
Dockerfiles included inside the image? We can avoid copying those, or any other
unneeded files into our image by creating a `.dockerignore` file:

```
echo 'Dockerfile*' >>.dockerignore
docker image build -f Dockerfile.multi-stage --target build -t my-hello:build .
```

Lets make sure the container still works and then look at the source directory:

```
docker container run -it --rm my-hello:build
docker container run -it --rm my-hello:build ls -l
```

## Build Args

There's been a request to try using different base images for the build for
testing. Run the same multi-stage build using an Alpine based base image. Lets
also use a build arg to inject the version into the image labels:

```
docker image build -f Dockerfile.multi-stage --target build -t my-hello:alpine \
  --build-arg GOLANG_VER=1.10-alpine --build-arg APPVER=1.0.0-alpine .
docker container run -it --rm my-hello:alpine
docker image inspect \
  --format '{{range $k,$v := .Config.Labels}}{{printf "%s = %s\n" $k $v}}{{end}}' \
  my-hello:alpine
```

Did the image still work with the version change? Labels are often used to add
additional data to objects in docker, which you can then query later. They can
be very useful for scripting, e.g. identifying images that can be automatically
purged.

Can you explain what's happening in the format string above? If you have extra
time, try to output different fields from the inspect output using the format
argument.


