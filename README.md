# 


## housekeeping

```bash
docker run -it -d -p 8080:8080 -v "/home/trapapa/playground/telegraf-influxdb-grafana-docker-composer:/home/coder/project" -u "$(id -u):$(id -g)" codercom/code-server:latest
git config --global user.name "Mathias Stadler"
git config --global user.EMAIL "email@mathias-stadler.de"
# https://help.github.com/en/github/using-git/caching-your-github-password-in-git
git config --global credential.helper 'cache --timeout=3600'
```


## source

```txt
# telegraf
https://hub.docker.com/_/telegraf
# influxDB
https://hub.docker.com/_/influxdb/
# grafana
https://hub.docker.com/r/grafana/grafana/

# docker access to system
https://www.jacobtomlinson.co.uk/monitoring/2016/06/23/running-telegraf-inside-a-container/

# 
https://github.com/influxdata/telegraf/issues/2112

```

## https://github.com/influxdata/telegraf/issues/2112

- based on ideas of

```txt
https://github.com/nicolargo/docker-influxdb-grafana/blob/master/docker-compose.yml
```

```yaml
influxdb:
  image: influxdb:latest
  container_name: influxdb
  ports:
    - "8083:8083"
    - "8086:8086"
    - "8090:8090"
  env_file:
    - 'env.influxdb'
  volumes:
    # Data persistency
    # sudo mkdir -p /srv/docker/influxdb/data
    - /srv/docker/influxdb/data:/var/lib/influxdb

telegraf:
  image: telegraf:latest
  container_name: telegraf
  links:
    - influxdb
  volumes:
    - ./telegraf.conf:/etc/telegraf/telegraf.conf:ro

grafana:
  image: grafana/grafana:latest
  container_name: grafana
  ports:
    - "3000:3000"
  env_file:
    - 'env.grafana'
  user: "0"
  links:
    - influxdb
  volumes:
    # Data persistency
    # sudo mkdir -p /srv/docker/grafana/data; chown 472:472 /srv/docker/grafana/data
    - /srv/docker/grafana/data:/var/lib/grafana
```


##  generate a default telegraf.conf from docker

```bash
docker run --rm telegraf:${IMAGE_TAG_TELEGRAF} telegraf config > telegraf.conf
```