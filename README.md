# EaseProbe

EaseProbe is a simple, standalone, and lightWeight tool that can do health/status checking, written in Go.

- [EaseProbe](#easeprobe)
  - [1. Overview](#1-overview)
    - [1.1 Probe](#11-probe)
    - [1.2 Notification](#12-notification)
    - [1.3 Report](#13-report)
  - [2. Getting Start](#2-getting-start)
    - [2.1 Build](#21-build)
    - [2.2 Run](#22-run)
  - [3. Configuration](#3-configuration)
    - [3.1 HTTP Probe Configuration](#31-http-probe-configuration)
    - [3.2 TCP Probe Configuration](#32-tcp-probe-configuration)
    - [3.4 Shell Command Probe Configuration](#34-shell-command-probe-configuration)
    - [3.5 Native Client Probe](#35-native-client-probe)
    - [3.6 Notification Configuration](#36-notification-configuration)
    - [3.7 Global Setting Configuration](#37-global-setting-configuration)
  - [4. Community](#4-community)
  - [5. License](#5-license)

## 1. Overview

EaseProbe would do 3 kinds of works - **Probe**, **Notify** and **Report**. 

### 1.1 Probe

Ease Probe supports the following probing methods:

- **HTTP**. Checking the HTTP status code, Support mTLS, HTTP Basic Auth, and can set the Request Header/Body. ( [HTTP Probe Configuration](#31-http-probe-configuration) )

  ```YAML
  http:
    # Some of the Software support the HTTP Query
    - name: ElasticSearch
      url: http://elasticsearch.server:9200
    - name: Prometheus
      url: http://prometheus:9090/graph
  ```

- **TCP**. Just simply check the TCP connection can be established or not. ( [TCP Probe Configuration](#32-tcp-probe-configuration) )

  ```YAML
  tcp:
    - name: Kafka
      host: kafka.server:9093
  ```

- **Shell**. Run a Shell command and check the result. ( [Shell Command Probe Configuration](#34-shell-command-probe-configuration) )

  ```YAML
  shell:
    # run redis-cli ping and check the "PONG"
    - name: Redis (Local)
      cmd: "redis-cli"
      args:
        - "-h"
        - "127.0.0.1"
        - "ping"
      env:
        # set the `REDISCLI_AUTH` environment variable for redis password
        - "REDISCLI_AUTH=abc123" 
      # check the command output, if does not contain the PONG, mark the status down
      contain : "PONG"
  ```

- **Client**. Currently, support the following native client. Support the mTLS. ( [Native Client Probe](#35-native-client-probe) )
  - **MySQL**. Connect to the MySQL server and run the `SHOW STATUS` SQL.
  - **Redis**. Connect to the Redis server and run the `PING` command.
  - **MongoDB**. Connect to MongoDB server and just ping server.
  - **Kafka**. Connect to Kafka server and list all topics.


  ```YAML
  client:
    - name: Kafka Native Client (local)
      driver: "kafka"
      host: "localhost:9093"
      # mTLS
      ca: /path/to/file.ca
      cert: /path/to/file.crt
      key: /path/to/file.key
  ```


### 1.2 Notification

Ease Probe supports the following notifications: 

- **Email**. Support multiple email addresses.
- **Slack**. Using Webhook for notification
- **Discord**. Using Webhook for notification
- **Log File**. Write the notification into a log file


```YAML
# Notification Configuration
notify:
  slack:
    - webhook: "https://hooks.slack.com/services/........../....../....../"
  discord:
    - webhook: "https://discord.com/api/webhooks/...../....../"
  email:
    - server: smtp.email.example.com:465
      username: user@example.com
      password: ********
      to: "user1@example.com;user2@example.com"
```

Check the  [Notification Configuration](#36-notification-configuration) to see how to configure it.

### 1.3 Report

- **SLA Report**. EaseProbe would send the daily, weekly or monthly SLA report. 

```YAML
settings:
  # SLA Report schedule
  sla:
    #  daily, weekly (Sunday), monthly (Last Day), none
    schedule : "weekly"
    # UTC time, the format is 'hour:min:sec'
    time: "23:59"
```


**Note**: 

- The notification is **Edge-Triggered Mode**, only notified while the status is changed.

## 2. Getting Start

### 2.1 Build

Compiler `Go 1.17+`

Use `make` to make the binary file. the target is under the `build/bin` directory

```shell
$ make
```

### 2.2 Run

Running the following command for local test

```shell
$ build/bin/easeprobe -f config.yaml 
```


## 3. Configuration

The following configuration is an example.

### 3.1 HTTP Probe Configuration

```YAML
# HTTP Probe Configuration

http:
  # A Website
  - name: MegaEase Website (Global)
    url: https://megaease.com

  # Some of the Software support the HTTP Query
  - name: ElasticSearch
    url: http://elasticsearch.server:9200
  - name: Eureka 
    url: http://eureka.server:8761
  - name: Prometheus
    url: http://prometheus:9090/graph

  # Spring Boot Application with Actuator Heath API
  - name: EaseService-Governance 
    url: http://easeservice-mgmt-governance:38012/actuator/health
  - name: EaseService-Control 
    url: http://easeservice-mgmt-control:38013/actuator/health
  - name: EaseService-Mesh
    url: http://easeservice-mgmt-mesh:38013/actuator/health

  # A completed HTTP Probe configuration
  - name: Special Website
    url: https://megaease.cn
    # Request Method
    method: GET
    # Request Header
    headers:
      X-head-one: xxxxxx
      X-head-two: yyyyyy
      X-head-THREE: zzzzzzX-
    content_encoding: text/json
    # Request Body
    body: '{ "FirstName": "Mega", "LastName" : "Ease", "UserName" : "megaease", "Email" : "user@example.com"}'
    # HTTP Basic Auth
    username: username
    password: password
    # mTLS
    ca: /path/to/file.ca
    cert: /path/to/file.crt
    key: /path/to/file.key
    # configuration
    timeout: 10s # default is 30 seconds
    interval: 60s # default is 60 seconds

```
### 3.2 TCP Probe Configuration

```YAML
# TCP Probe Configuration
tcp:
  - name: SSH Service
    host: example.com:22
    timeout: 10s # default is 30 seconds
    interval: 2m # default is 60 seconds

  - name: Kafka
    host: kafka.server:9093
```

### 3.4 Shell Command Probe Configuration

```YAML
# Shell Probe Configuration
shell:
  # A proxy curl shell script
  - name: Google Service
    cmd: "./resources/probe/scripts/proxy.curl.sh" 
    args:
      - "socks5://127.0.0.1:1085"
      - "www.google.com"

  # run redis-cli ping and check the "PONG"
  - name: Redis (Local)
    cmd: "redis-cli"
    args:
      - "-h"
      - "127.0.0.1"
      - "ping"
    env:
      # set the `REDISCLI_AUTH` environment variable for redis password
      - "REDISCLI_AUTH=abc123" 
    # check the command output, if does not contain the PONG, mark the status down
    contain : "PONG"

  # Run Zookeeper command `stat` to check the zookeeper status
  - name: Zookeeper (Local)
    cmd: "/bin/sh"
    args:
      - "-c"
      - "echo stat | nc 127.0.0.1 2181"
    contain: "Mode:"
```

### 3.5 Native Client Probe

```YAML
# Native Client Probe
client:
  - name: Redis Native Client (local)
    driver: "redis"  # driver is redis
    host: "localhost:6379"  # server and port
    password: "abc123" # password
    # mTLS
    ca: /path/to/file.ca
    cert: /path/to/file.crt
    key: /path/to/file.key

  - name: MySQL Native Client (local)
    driver: "mysql"
    host: "localhost:3306"
    username: "root"
    password: "pass"

  - name: MongoDB Native Client (local)
    driver: "mongo"
    host: "localhost:27017"
    username: "admin"
    password: "abc123"
    timeout: 5s

  - name: Kafka Native Client (local)
    driver: "kafka"
    host: "localhost:9093"
    # mTLS
    ca: /path/to/file.ca
    cert: /path/to/file.crt
    key: /path/to/file.key
```


### 3.6 Notification Configuration

```YAML
# Notification Configuration
notify:
  # Notify to a local log file
  log:
    - file: "/tmp/easeprobe.log"
      dry: true  
  # Notify to Slack Channel
  slack:
    - webhook: "https://hooks.slack.com/services/........../....../....../"
      # dry: true   # dry notification, print the Slack JSON in log(STDOUT)
  # Notify to Discord Text Channel
  discord:
    - webhook: "https://discord.com/api/webhooks/...../....../"
      # dry: true # dry notification, print the Discord JSON in log(STDOUT)
      retry: # something the network is not good need to retry.
        times: 3
        interval: 10s
  email:
    - server: smtp.email.example.com:465
      username: user@example.com
      password: ********
      to: "user1@example.com;user2@example.com"
      # dry: true # dry notification, print the Email HTML in log(STDOUT)
```


### 3.7 Global Setting Configuration

```YAML
# Global settings for all probes and notifiers.
settings:
  # SLA Report schedule
  sla:
    #  daily, weekly (Sunday), monthly (Last Day), none
    schedule : "daily"
    # UTC time, the format is 'hour:min:sec'
    time: "23:59"

  notify:
    # dry: true # Global settings for dry run 
    retry: # Global settings for retry 
      times: 5
      interval: 10s

  probe:
    timeout: 30s # the time out for all probes
    interval: 1m # probe every minute for all probes
  # easeprobe program running log file.
  logfile: "test.log" 

  # Log Level Configuration
  # can be: panic, fatal, error, warn, info, debug.
  loglevel: "debug"

  # debug mode 
  # - true: send the SLA report every minute
  # - false: send the SLA report in schedule
  debug: false
 
  # Date format
  # Date
  #  - January 2, 2006
  #  - 01/02/06
  #  - Jan-02-06
  #
  # Time
  #   - 15:04:05
  #   - 3:04:05 PM
  #
  # Date Time
  #   - Jan _2 15:04:05                   (Timestamp)
  #   - Jan _2 15:04:05.000000            (with microseconds)
  #   - 2006-01-02T15:04:05-0700          (ISO 8601 (RFC 3339))
  #   - 2006-01-02 15:04:05
  #   - 02 Jan 06 15:04 MST               (RFC 822)
  #   - 02 Jan 06 15:04 -0700             (with numeric zone)
  #   - Mon, 02 Jan 2006 15:04:05 MST     (RFC 1123)
  #   - Mon, 02 Jan 2006 15:04:05 -0700   (with numeric zone)
  timeformat: "2006-01-02 15:04:05 UTC"

```

## 4. Community

- [Join Slack Workspace](https://join.slack.com/t/openmegaease/shared_invite/zt-upo7v306-lYPHvVwKnvwlqR0Zl2vveA) for requirement, issue and development.
- [MegaEase on Twitter](https://twitter.com/megaease)

## 5. License 

EaseProbe is under the Apache 2.0 license. See the [LICENSE](./LICENSE) file for details.