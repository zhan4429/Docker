
## docker buildx
To build Docker images for both `arm64` and `amd64` architectures on a MacBook M1, you can use Docker's `buildx` feature. Here are the steps:

1. First, ensure that Docker Desktop is installed and running on your MacBook M1.

2. Open a terminal and create a new builder instance that enables multi-architecture builds:

```bash
docker buildx create --use --name mybuilder
```

3. Boot up the builder instance:

```bash
docker buildx inspect --bootstrap
```

4. Now you can build your Docker image for both `arm64` and `amd64` architectures. Replace `<path_to_dockerfile>` with the path to your Dockerfile, and `<image_name>` with the name you want to give to your Docker image:

```bash
docker buildx build --platform linux/amd64,linux/arm64 -t <image_name> <path_to_dockerfile> --push
```

Note: The `--push` flag is used to push the image to Docker Hub. If you don't want to push the image, you can use `--load` instead, but note that `--load` only works for single platform images.

Remember to replace `<path_to_dockerfile>` with the path to your Dockerfile, and `<image_name>` with the name you want to give to your Docker image.