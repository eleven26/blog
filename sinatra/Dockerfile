FROM ubuntu:16.04
MAINTAINER James Turnbull "james@example.com"
ENV REFRESHED_AT 2014-06-01
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update -yqq && apt-get -yqq install ruby2.3 ruby2.3-dev build-essential redis-tools
RUN gem install --no-rdoc --no-ri sinatra json redis

RUN mkdir -p /opt/webapp

EXPOSE 4567

CMD [ "/opt/webapp/bin/webapp" ]
