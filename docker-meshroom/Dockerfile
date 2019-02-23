FROM nvidia/cuda:10.0-base

RUN apt-get update && apt-get install -y \
    wget make unzip libgssapi-krb5-2 \
    && rm -rf /var/lib/apt/lists/*

RUN wget https://github.com/alicevision/meshroom/releases/download/v2018.1.0/Meshroom-2018.1.0-linux.tar.gz;\
    tar -xzvf Meshroom-2018.1.0-linux.tar.gz;\
    mv Meshroom-2018.1.0 /opt/meshroom;\
    rm Meshroom-2018.1.0-linux.tar.gz
ENV PATH="/opt/meshroom:/opt/meshroom/aliceVision/bin:${PATH}"

CMD ["make"]