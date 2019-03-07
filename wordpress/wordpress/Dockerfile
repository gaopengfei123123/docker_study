FROM wordpress:latest
ADD sources.list /etc/apt/sources.list
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys \
    && apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 3B4FE6ACC0B21F32 
RUN echo "export TERM=xterm LANG=en_US.UTF-8" >> ~/.bashrc \
    && apt-get update && apt-get -y install git \
    && rm -rf "/opt/letsencrypt" \
    && git clone https://github.com/letsencrypt/letsencrypt /opt/letsencrypt \
    && cd /opt/letsencrypt \
    && ./letsencrypt-auto --version