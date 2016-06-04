FROM ubuntu:14.04

RUN apt-get update -y
RUN apt-get install -y git bash-completion dnsutils nginx telnet lsof
RUN git clone https://github.com/letsencrypt/letsencrypt
RUN cd letsencrypt && ./letsencrypt-auto --help

EXPOSE 443
# WORKDIR ~

CMD bash

