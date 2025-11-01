# Tree Demo with NanoOwl
## Create Dockerfile

```bash
sudo vi Dockerfile
```

```Dockerfile
FROM dustynv/nanoowl:r36.4.0

LABEL maintainer="changxubo"
LABEL description="Nanoowl real-time detection and classification for anthing"

WORKDIR /opt/nanoowl/examples/tree_demo

RUN pip3 install --no-cache-dir aiohttp --index-url=https://pypi.org/simple

EXPOSE 7860

ENV CAMERA_ID=4
ENV RESOLUTION=640x480

ENTRYPOINT ["python3","tree_demo.py","../../data/owl_image_encoder_patch32.engine","--camera","${CAMERA_ID}","--resolution","${RESOLUTION}"]

```

## Build Docker Image
```bash
sudo docker build . -t dustynv/nanoowl:r36.4.1
```

## Run Docker Container

Modify run.sh with 'docker run -d --restart unless-stopped -id' to run container in background and restart automatically

```bash
jetson-containers run dustynv/nanoowl:r36.4.1

docker logs jetson_container_20251030_161500 -f  # to view logs
```

## Health Check
```bash
docker ps -a  # check all containers
docker logs jetson_container_20251030_161500 -f  # to view logs
docker restart jetson_container_20251030_161500  # start container if not running
``` 
