## Logstash配置文件
> 适配Spring Boot默认的日志格式

```
input {
  file {
    type => "log4j"
    path => "/home/curry/logger/logs/*.log"
  }
}
filter {
  multiline {
    pattern => "^\d{4}-\d{1,2}-\d{1,2}\s\d{1,2}:\d{1,2}:\d{1,2}"
    negate => true
    what => "previous"
  }
  grok {
    match => ["message", "%{TOMCATLEGACY_DATESTAMP:datetime}\s+%{LOGLEVEL:level}\s+%{NUMBER:pid}\s+---\s+\[\s*%{NOTSPACE:thread-name}\s*\]\s+%{NOTSPACE:class-name}\s+:\s+%{GREEDYDATA:details}"]
  }
}
output {
  elasticsearch {
    hosts => ["https://10.211.55.12:9200"]
    cacert => "/home/curry/logger/logstash-8.5.3/certs/http_ca.crt"
    index => "logger-02"
    user => "elastic"
    password => "_mmpi+iBNlZgodfOxyZC"
  }
}
```

> 如果没有multiline插件，需要手动安装
> 
> ```
> logstash-plugin install logstash-filter-multiline
> ```


**自带的正则匹配**
> https://github.com/logstash-plugins/logstash-patterns-core
>
> 在patterns目录下已经包含了120种正则表达式