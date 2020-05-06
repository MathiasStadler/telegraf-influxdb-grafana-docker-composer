# telegraf-influxdb-grafana-docker-composer


## TL;DR just for test **NOT** fit for productiuon

```bash
cd && \
git clone  https://github.com/MathiasStadler/telegraf-influxdb-grafana-docker-composer.git && \
cd telegraf-influxdb-grafana-docker-composer && \
certtool --generate-privkey --outfile server-key.pem --bits 2048 && \
certtool --generate-self-signed --load-privkey server-key.pem --outfile server-cert.pem --template cert.cfg && \
docker-compose -f tig-compose.yml up -d && \
docker-compose -f tig-compose.yml logs -f

# open grafana
http://localhost:3000

# down and remove nwtworks and voulmes
docker-compose -f tig-compose.yml down -v
```

## housekeeping

```bash
docker run -it -d -p 8080:8080 -v "/home/trapapa/playground/telegraf-influxdb-grafana-docker-composer:/home/coder/project" -u "$(id -u):$(id -g)" codercom/code-server:latest --cert
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
#
https://github.com/infcatluxdata/influxdata-docker/blob/master/influxdb/1.8/init-influxdb.sh
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

##  generate a default telegraf.conf from docker

```bash
# set docker image tag
IMAGE_TAG_TELEGRAF=$(grep IMAGE_TAG_TELEGRAF .env |sed 's/.*=//')
echo "IMAGE_TAG_TELEGRAF => $IMAGE_TAG_TELEGRAF"
docker run --rm telegraf:${IMAGE_TAG_TELEGRAF} telegraf config > telegraf.conf
```

## export IMAGE_TAG_INFLUXDB to bash

```bash
export IMAGE_TAG_INFLUXDB=$(grep IMAGE_TAG_INFLUXDB .env |sed 's/.*=//')
echo "IMAGE_TAG_INFLUXDB => $IMAGE_TAG_INFLUXDB"
docker run --rm influxdb:${IMAGE_TAG_INFLUXDB} influxd config > influxdb.conf
```

## start influx cli in bash

```bash
export IMAGE_TAG_INFLUXDB=$(grep IMAGE_TAG_INFLUXDB .env |sed 's/.*=//')
echo "IMAGE_TAG_INFLUXDB => $IMAGE_TAG_INFLUXDB"
docker exec -it influxdb influx -ssl -unsafeSsl -username 'admin' -password 'admin'  -database 'telegraf'
```

## execute query from cli via docker

```bash
# with admin user
docker exec -it influxdb influx -ssl -unsafeSsl -username 'admin' -password 'admin'  -database 'telegraf' -execute 'SELECT "host","cpu","usage_idle" FROM cpu WHERE time > now() - 60s   '
# with datssource user
docker exec -it influxdb influx -ssl -unsafeSsl -username 'telegraf_read' -password 'telegraf_read'  -database 'telegraf' -execute 'SELECT "host","cpu","usage_idle" FROM cpu WHERE time > now() - 60s '
# get all series of telegraf database
docker exec -it influxdb influx -ssl -unsafeSsl -username 'admin' -password 'admin'  -database 'telegraf' -execute 'show series'
```

## execute influx from cli 

```bash
docker exec -it influxdb influx -ssl -unsafeSsl -username 'admin' -password 'admin'
```

- sample query **IN** influx

```bash
## show series of db
# login with influx last step
# use <databases>
use telegraf
show series
# select * from <series> WHERE time > now() - 30s
select * from cpu WHERE time > now() - 30s
```


## influxdb HTTP API

-  get all databases name

```bash
# without aurg agains database
curl -G "http://somehost:8086/query?pretty=true" --data-urlencode "q=show databases"

# with auth with datasource user
# this must work for grafana
curl -k  -G 'https://localhost:8086/query?db=telegraf' -u telegraf_read:telegraf_read  --data-urlencode "q=SELECT "host","cpu","usage_idle" FROM cpu WHERE time > now() - 60s"

# admin user
curl -k  -G 'https://localhost:8086/query?db=telegraf' -u admin:admin  --data-urlencode "q=SELECT "host","cpu","usage_idle" FROM cpu WHERE time > now() - 60s"
```

## influxdb with **https**

- https://www.gnutls.org/manual/html_node/certtool-Invocation.html

## install gnutls-bin

```bash
# ubuntu
sudo apt-get install -y gnutls-bin
```

## create private server key for influxdb server

- HINT: no as root

```bash
certtool --generate-privkey --outfile server-key.pem --bits 2048
```

## Create a cert for InfluxDB server

```bash
certtool --generate-self-signed --load-privkey server-key.pem --outfile server-cert.pem
```
- or non interactive 
- with parameter from template filecert.cfg

```bash
certtool --generate-self-signed --load-privkey server-key.pem --outfile server-cert.pem --template cert.cfg 
```

## chmod keys
- @TODO check obsolete 
```bash
 sudo chown influxdb:influxdb server-key.pem server-cert.pem
 @TODO determine the user if of influxdb
 sudo chown 999:999 server-key.pem server-cert.pem
```

## ERROR message at start 

- message

```txt
influxdb    | run: open server: open service: tls: failed to find any PEM data in certificate input
````

- solution 

create keys new and check the size of it


## config influxdb.conf for https

@TODO how 
- create first a default config

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
# urls = ["https://127.0.0.1:8086"]
# set by env variable
urls = ["$INFLUXDB_URI"] # required

## Use TLS but skip chain & host verification
insecure_skip_verify = true
```

## grafana get all data sources

```text
https://grafana.com/docs/grafana/latest/http_api/data_source/
```
https://grafana.com/docs/grafana/latest/administration/provisioning/

## better source
https://github.com/grafana/grafana/blob/master/docs/sources/administration/provisioning.md

## sample datasource

```yaml
# config file version
apiVersion: 1

# list of datasources that should be deleted from the database
deleteDatasources:
  - name: Graphite
    orgId: 1

# list of datasources to insert/update depending
# what's available in the database
datasources:
  # <string, required> name of the datasource. Required
- name: Graphite
  # <string, required> datasource type. Required
  type: graphite
  # <string, required> access mode. proxy or direct (Server or Browser in the UI). Required
  access: proxy
  # <int> org id. will default to orgId 1 if not specified
  orgId: 1
  # <string> custom UID which can be used to reference this datasource in other parts of the configuration, if not specified will be generated automatically
  uid: my_unique_uid
  # <string> url
  url: http://localhost:8080
  # <string> Deprecated, use secureJsonData.password
  password:
  # <string> database user, if used
  user:
  # <string> database name, if used
  database:
  # <bool> enable/disable basic auth
  basicAuth:
  # <string> basic auth username
  basicAuthUser:
  # <string> Deprecated, use secureJsonData.basicAuthPassword
  basicAuthPassword:
  # <bool> enable/disable with credentials headers
  withCredentials:
  # <bool> mark as default datasource. Max one per org
  isDefault:
  # <map> fields that will be converted to json and stored in jsonData
  jsonData:
     graphiteVersion: "1.1"
     tlsAuth: true
     tlsAuthWithCACert: true
  # <string> json object of data that will be encrypted.
  secureJsonData:
    tlsCACert: "..."
    tlsClientCert: "..."
    tlsClientKey: "..."
    # <string> database password, if used
    password:
    # <string> basic auth password
    basicAuthPassword:
  version: 1
  # <bool> allow users to edit datasources from the UI.
  editable: false
```

## TICK stack

https://docs.influxdata.com/



curl -k  -G 'https://localhost:8086/query?db=telegraf' -u telegraf_read:telegraf_read  --data-urlencode "q=SELECT "host","cpu","usage_idle" FROM cpu WHERE time > now() - 60s"



# via url
curl -k  -G 'https://localhost:8086/query?db=telegraf' -u admin:admin  --data-urlencode "q=SELECT "host","cpu","usage_idle" FROM cpu WHERE time > now() - 60s"


docker exec -it influxdb influx -ssl -unsafeSsl -username 'admin' -password 'admin'  -database 'telegraf' -execute 'SELECT distinct("cpu") from (SELECT "cpu","host","usage_idle" FROM cpu WHERE time > now() - 20s GROUP BY *)' -format 'csv' --pretty

https://stackoverflow.com/questions/39216606/how-to-use-distinct-function-in-influxdb


SHOW TAG VALUES FROM "cpu" WITH KEY = "cpu"

https://grafana.com/grafana/dashboards/928



## make telegram from source

- clone

```bash
cd && cd playground
git clone https://github.com/influxdata/telegraf.git
cd telegraf 
make -j 4
```

- update go 

```bash
# from here https://forum.golangbridge.org/t/go-unknown-subcommand-mod/15257/2 
    First remove apt installed go : $ sudo apt-get remove go
    Download the tar file for linux from the site https://golang.org/dl/ 719
    Execute the command $ sudo tar -C /usr/local -xzf
    In the file ~/.bash_aliases add the following lines:
    export GOROOT="/usr/local/go"
    export GOBIN="$HOME/go/bin"
    mkdir -p $GOBIN
    export PATH=$PATH:$GOROOT/bin:$GOBIN

```

docker run --rm -v "$PWD":/usr/src/myapp -w /usr/src/myapp golang:1.13 make

sudo tcpdump -i wlp3s0 port 8086