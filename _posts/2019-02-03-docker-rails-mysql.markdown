---
layout: post
title:  "Docker로 Rails, MySQL개발 환경 설정"
date:   2019-02-03 20:35
author: lostfind
categories: Development
tags:	Rails Docker
cover:  "/assets/instacode.png"
---

Docker에서 Rails로 웹개발을 하기 위해 세팅했던 내용들

``` dockerfile:./docker/mysql/Dockerfile
FROM mysql:5.7.25
RUN apt-get update && apt-get install -y apt-utils locales && \
    rm -rf /var/lib/apt/lists/* && echo "ja_JP.UTF8 UTF-8" > /etc/locale.gen && \
    locale-gen ja_JP.UTF-8
ENV LC_ALL ja_JP.UTF-8
ADD ./docker/mysql/my.cnf /etc/my.cnf
```

```dockerfile:./docker/ruby/Dockerfile
FROM ruby:2.6.1
ENV LANG C.UTF-8

#for MySQL
RUN apt-get update -qq && apt-get install -y apt-utils \
      build-essential libpq-dev nodejs mysql-client

RUN gem install bundler

WORKDIR /tmp
ADD Gemfile Gemfile
ADD Gemfile.lock Gemfile.lock
RUN bundle install

ENV APP_HOME /accountbook
RUN mkdir -p $APP_HOME
WORKDIR $APP_HOME
ADD . $APP_HOME
```


```yml:docker-compose.yml
version: '2'
services:
  datastore:
    image: busybox
    volumes:
      - /share
      - ./docker/mysql/volumes:/var/lib/mysql
  server:
    build:
      context: .
      dockerfile: ./docker/nginx/Dockerfile
    ports:
      - '8090:80'
    volumes_from:
      - datastore
    depends_on:
      - datastore
  web:
    build:
      context: .
      dockerfile: ./docker/ruby/Dockerfile
    command: bundle exec rails s -p 3000 -b '0.0.0.0'
    ports:
      - '3600:3000'
    volumes:
      - .:/accountbook
    environment:
      RAILS_ENV: development
    volumes_from:
      - datastore
    depends_on:
      - db
    links:
      - db
      - db:database
      - db:mysql
    extends:
      file: ./docker/mysql/password.yml
      service: password
  db:
    build:
      context: .
      dockerfile: ./docker/mysql/Dockerfile
    ports:
      - '3606:3306'
    volumes_from:
      - datastore
    depends_on:
      - datastore
    extends:
      file: ./docker/mysql/password.yml
      service: password
```

### 레일즈 초기 설정
```sh
$ docker-compose run --rm web rails new . --force --database=mysql --skip-bundle
```

### DB 초기설정 (레일즈 이용)
```sh
$ docker-compose run --rm web bundle exec rake db:create
```

## 트러블
레일즈가 제대로 시작되지 않을 때

```sh
$ rm tmp/pids/server.pid
```

## docker-compose 관련 명령어
```sh
# 중지
$ docker-compose down

# 실행
$ docker-compose up

# 실행 (데몬, 백그라운드)
$ docker-compose up

# 현재 상태 확인
$ docker-compose ps
```