# logspout-logstash

Tiny [Logspout](https://github.com/gliderlabs/logspout) adapter to send Docker container logs to [Logstash](https://github.com/elastic/logstash).

## Example

A sample `docker-compose.yaml` file:

```yaml
version: '3.4'
services:
  logspout:
    image: waliot/logspout-logstash:latest
    environment:
      ROUTE_URIS: logstash://logstash:5000
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock

  logstash:
    image: logstash:latest
    command: -f /opt/logstash/sample.conf
    volumes:
      - ./logstash/:/opt/logstash

  # This is just an example.
  # Normally you would put your own services in this file.
  # Similar setup works on Kubernetes as well.
  redis:
    image: redis:latest
```

A sample Logstash configuration `logstash/sample.conf`:

```conf
input {
  udp {
    port  => 5000
    codec => json
  }
}

filter {
  if [docker][image] =~ /^logstash/ {
    drop { }
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
  }
  stdout { codec => rubydebug }
}
```
