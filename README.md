fluentd-pilot-forwarder
=============

[`fluentd-pilot`](https://github.com/AliyunContainerService/fluentd-pilot/blob/master/README.md)is an awesome docker log tool. With [fluentd-pilot](https://github.com/AliyunContainerService/fluentd-pilot/blob/master/README.md) you can collect logs from docker hosts and send them to your centralize log system such as elasticsearch, graylog2, awsog and etc. [fluentd-pilot](https://github.com/AliyunContainerService/fluentd-pilot/blob/master/README.md)  can collect not only docker stdout but also log file that inside docker containers.

Before start, read this.
[fluentd-docker-logging](http://docs.fluentd.org/v0.12/articles/docker-logging)


Try it
======

Prerequisites:

- docker-compose >= 1.6
- Docker Engine >= 1.10
- [Fluentd](http://fluentd.org/)>= 0.12 

``` yaml
version: '3'
services:
  pilot:
    image: 'arkii/fluentd-pilot:latest'
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /:/host
    environment:
      FLUENTD_HOST: '10.30.57.145'
      FLUENTD_OUTPUT: fluentd_forward
    deploy:
      mode: global
    restart:  always
```


Install Fluentd-aggragator
========

[install-by-rpm](http://docs.fluentd.org/v0.12/articles/install-by-rpm)


```
fluent-gem install fluent-plugin-file-alternative
fluent-gem install fluent-plugin-forest
fluent-gem install fluent-plugin-record-reformer
```

``` 
# cat /data/fluentd/etc/fluent.conf
<source>
  @type  forward
  bind 0.0.0.0
  port  24224
</source>

<filter **>
  @type stdout
</filter>

<match docker.**>
  type record_reformer
  remove_keys host,@target,docker_app,stream,@timestamp
  renew_record false
  enable_ruby false
  tag reformed.${tag_parts[0]}.${record["docker_app"]}.${tag_parts[2]}
</match>

<match reformed.docker.**>
  type forest
  subtype file
  <template>
    type file_alternative
    output_time false
    output_tag false
    add_newline false
    path /srv/log/${tag_parts[2]}/${tag_parts[3]}-*.log
    time_slice_format %m-%d
  </template>
</match>
```