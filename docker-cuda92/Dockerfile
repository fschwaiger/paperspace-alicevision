FROM nvidia/cuda:9.2-base

RUN apt-get update && apt-get install -y wget make unzip && rm -rf /var/lib/apt/lists/*

RUN wget https://github.com/alicevision/meshroom/releases/download/v2018.1.0/Meshroom-2018.1.0-linux.tar.gz;\
    tar -xzvf Meshroom-2018.1.0-linux.tar.gz;\
    mv Meshroom-2018.1.0/aliceVision /opt/aliceVision;\
    rm Meshroom-2018.1.0-linux.tar.gz;\
    rm -r Meshroom-2018.1.0
ENV PATH="/opt/aliceVision/bin:${PATH}"

CMD ["make"]