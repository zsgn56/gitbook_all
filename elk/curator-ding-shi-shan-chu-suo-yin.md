1、安装curator

```
pip install -U elasticsearch-curator
```

2、配置config.yml

```
---
client:
  hosts:
    - 192.168.100.199
  port: 9200
  url_prefix:
  use_ssl: False
  certificate:
  client_cert:
  client_key:
  ssl_no_validate: False
  http_auth:
  timeout: 3000
  master_only: False

logging:
  loglevel: INFO
  logfile: /var/log/curator.log
  logformat: default
  blacklist: ['elasticsearch', 'urllib3']
```

3、配置action.yml

```
---
actions:
  1:
    action: delete_indices
    description: >-
      show indices older than 30 days (based on index name)
    options:
      ignore_empty_list: True
      timeout_override:
      continue_if_exception: False
      disable_action: False
    filters:
    - filtertype: pattern
      kind: prefix
      value: logstash-application
      exclude:
    - filtertype: age
      source: name
      direction: older
      timestring: '%Y%m%d'
      unit: days
      unit_count: 30
      exclude:
```

4、执行命令

```
curator --config config.yml action.yml
```



