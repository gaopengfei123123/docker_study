FROM ubuntu:14.04
MAINTAINER gaofeifiy/5173180@qq.com
RUN cp /etc/apt/sources.list /etc/apt/sources.list.bak
RUN rm /etc/apt/sources.list
ADD sources.list /etc/apt/sources.list
RUN ["apt-get","update"]
RUN ["apt-get","install","-y","vim"]
RUN ["apt-get","install","-y","wget"]
RUN wget http://nginx.org/keys/nginx_signing.key
RUN apt-key add nginx_signing.key
RUN echo "deb http://nginx.org/packages/ubuntu/ trusty nginx" >> /etc/apt/sources.list
RUN echo "deb-src http://nginx.org/packages/ubuntu/ trusty nginx" >> /etc/apt/sources.list
RUN apt-get update
RUN ["apt-get","install","-y","nginx"]
EXPOSE 8888
EXPOSE 2222
