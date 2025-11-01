FROM dustynv/nanoowl:r36.4.0

LABEL maintainer="changxubo"
LABEL description="Nanoowl real-time detection and classification for anthing"

WORKDIR /opt/nanoowl/examples/tree_demo

RUN pip3 install --no-cache-dir aiohttp --index-url=https://pypi.org/simple

EXPOSE 7860

ENV CAMERA_ID=4
ENV RESOLUTION=640x480

ENTRYPOINT ["python3","tree_demo.py","../../data/owl_image_encoder_patch32.engine","--camera","${CAMERA_ID}","--resolution","${RESOLUTION}"]

