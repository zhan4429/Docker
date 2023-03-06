Basically, a layer, or image layer is a change on an image, or an intermediate image. Every command you specify (FROM, RUN, COPY, etc.) in your Dockerfile causes the previous image to change, thus creating a new layer. You can think of it as staging changes when you're using git: You add a file's change, then another one, then another one...

Consider the following Dockerfile:

```
FROM rails:onbuild
ENV RAILS_ENV production
ENTRYPOINT ["bundle", "exec", "puma"]
```

First, we choose a starting image: `rails:onbuild`, which in turn has many layers. We add another layer on top of our starting image, setting the environment variable `RAILS_ENV` with the `ENV` command. Then, we tell docker to run `bundle exec puma` (which boots up the rails server). That's another layer.

The concept of layers comes in handy at the time of building images. Because layers are intermediate images, if you make a change to your Dockerfile, docker will rebuild only the layer that was changed and the ones after that. This is called layer caching.

If you change or add a layer, Docker will also build any layers that come afterwards because they might be affected by the change. 

A docker container image is created using a dockerfile. Every line in a dockerfile will create a layer. Consider the following dummy example:
```
FROM ubuntu             #This has its own number of layers say "X"
MAINTAINER FOO          #This is one layer 
RUN mkdir /tmp/foo      #This is one layer 
RUN apt-get install vim #This is one layer 
```

This will create a final image where the total number of layers will be X+3.


### Why layers?

The layers have a couple advantages. First, they are immutable. Once created, that layer identified by a sha256 hash will never change. That immutability allows images to safely build and fork off of each other. If two dockerfiles have the same initial set of lines, and are built on the same server, they will share the same set of initial layers, saving disk space. That also means if you rebuild an image, with just the last few lines of the Dockerfile experiencing changes, only those layers need to be rebuilt and the rest can be reused from the layer cache. This can make a rebuild of docker images very fast.

Inside a container, you see the image filesystem, but that filesystem is not copied. On top of those image layers, the container mounts it's own read-write filesystem layer. Every read of a file goes down through the layers until it hits a layer that has marked the file for deletion, has a copy of the file in that layer, or the read runs out of layers to search through. Every write makes a modification in the container specific read-write layer.

### Reducing layer bloat
One downside of the layers is building images that duplicate files or ship files that are deleted in a later layer. The solution is often to merge multiple commands into a single RUN command. Particularly when you are modifying existing files or deleting files, you want those steps to run in the same command where they were first created. A rewrite of the above Dockerfile would look like:

```
FROM busybox

RUN mkdir /data \
 && dd if=/dev/zero bs=1024 count=1024 of=/data/one \
 && chmod -R 0777 /data \
 && dd if=/dev/zero bs=1024 count=1024 of=/data/two \
 && chmod -R 0777 /data \
 && rm /data/one

CMD ls -alh /data
```
And if you compare the resulting images:

busybox: ~1MB
first image: ~6MB
second image: ~2MB
Just by merging together some lines in the contrived example, we got the same resulting content in our image, and shrunk our image from 5MB to just the 1MB file that you see in the final image.

