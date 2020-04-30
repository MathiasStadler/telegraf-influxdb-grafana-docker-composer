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
# manual
https://docs.influxdata.com/influxdb/v1.8/administration/authentication_and_authorization/
# grafana
https://hub.docker.com/r/grafana/grafana/

# docker access to system
https://www.jacobtomlinson.co.uk/monitoring/2016/06/23/running-telegraf-inside-a-container/

# Run telegraf inside docker, how to collect net, processors input in the host? #2112
https://github.com/influxdata/telegraf/issues/2112
#
https://devconnected.com/how-to-setup-telegraf-influxdb-and-grafana-on-linux/
```

## validate docker-compose.yml file

```bash
/usr/local/bin/docker-compose -f docker-compose.yml config
```

## find out which port used

```bash
netstat -ltnp | grep -w ':80' 
sudo netstat -pna | grep 3000
```

## https://github.com/influxdata/telegraf/issues/2112

- based on ideas of

```txt
https://github.com/nicolargo/docker-influxdb-grafana/blob/master/docker-compose.yml
#
https://github.com/jkehres/docker-compose-influxdb-grafana/blob/master/docker-compose.yml
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
# set docker image tag
IMAGE_TAG_TELEGRAF=$(grep IMAGE_TAG_TELEGRAF .env |sed 's/.*=//')
echo "IMAGE_TAG_TELEGRAF => $IMAGE_TAG_TELEGRAF"
docker run --rm telegraf:${IMAGE_TAG_TELEGRAF} telegraf config > telegraf.conf
```

##

```bash
IMAGE_TAG_INFLUXDB=$(grep IMAGE_TAG_INFLUXDB .env |sed 's/.*=//')
echo "IMAGE_TAG_INFLUXDB => $IMAGE_TAG_INFLUXDB"
docker run --rm influxdb:${IMAGE_TAG_INFLUXDB} influxd config > influxdb.conf
```

## start influx cli

```bash
IMAGE_TAG_TELEGRAF=$(grep IMAGE_TAG_TELEGRAF .env |sed 's/.*=//')

```

## influxdb HTTP API

-  get all databases name

```bash
curl -G "http://somehost:8086/query?pretty=true" --data-urlencode "q=show databases"
```

- get series from database

```bash
docker exec -it influxdb influx -ssl -unsafeSsl -username 'admin' -password 'admin'  -database 'telegraf' -execute 'show series'
```



https://github.com/infcatluxdata/influxdata-docker/blob/master/influxdb/1.8/init-influxdb.sh


## **https**

## install gnutls-bin

```bash
sudo apt-get install gnutls-bin
```

## create server key

```bash
sudo certtool --generate-privkey --outfile server-key.pem --bits 2048
```

## Create a public key for your InfluxDB server

```bash
 sudo certtool --generate-self-signed --load-privkey server-key.pem --outfile server-cert.pem
```

## chmod keys

```bash
 sudo chown influxdb:influxdb server-key.pem server-cert.pem
 @TODO determine the user if of influxdb
 sudo chown 999:999 server-key.pem server-cert.pem
```

## config /etc/influxdb/influxdb.conf for https

```bash
# Determines whether HTTPS is enabled.
  https-enabled = true

# The SSL certificate to use when HTTPS is enabled.
https-certificate = "/etc/ssl/influxdb/server-cert.pem"

# Use a separate private key location.
https-private-key = "/etc/ssl/influxdb/server-key.pem"
```

## Configure Telegraf for HTTPS

```bash
# Configuration for sending metrics to InfluxDB
[[outputs.influxdb]]

# https, not http!
urls = ["https://127.0.0.1:8086"]

## Use TLS but skip chain & host verification
insecure_skip_verify = true
```

## coonect influxdb via influx

```bash
docker exec -it influxdb influx -ssl -unsafeSsl -username 'admin' -password 'admin'
```

## show series of db

```bash
# login with influx last step
# use <databases>
use telegraf
show series
# select * from <series> WHERE time > now() - 30s
select * from cpu WHERE time > now() - 30s
```

