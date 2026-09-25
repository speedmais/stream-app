FROM ubuntu:20.04

RUN apt-get update && apt-get install -y \
    build-essential \
    wget \
    curl \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /tmp

RUN wget http://nginx.org/download/nginx-1.25.1.tar.gz && \
    tar -xzf nginx-1.25.1.tar.gz && \
    wget https://github.com/arut/nginx-rtmp-module/archive/master.zip && \
    apt-get install -y unzip && \
    unzip master.zip

WORKDIR /tmp/nginx-1.25.1

RUN ./configure --add-module=../nginx-rtmp-module-master && \
    make && \
    make install

RUN mkdir -p /tmp/rtmp/hls /tmp/rtmp/recordings

COPY nginx.conf /usr/local/nginx/conf/nginx.conf

EXPOSE 1935 8080

CMD ["/usr/local/nginx/sbin/nginx", "-g", "daemon off;"]
