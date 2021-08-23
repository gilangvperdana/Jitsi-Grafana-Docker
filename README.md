# Dockerize Jitsi Meet on VPS with Grafana for Monitoring
    Learn a little about Realm of public services (Jitsi).
    This project has been tested on a Linux server in a public cloud.
    Indonesian Tutorial : [https://gilangvperdana.medium.com/instalasi-jitsi-meet-pada-vps-bersama-grafana-b39ec22ba402]
```Environment:
Environment:
    1. Ubuntu21.04 Server
    2. 4GB RAM
    3. 48GB Storage
    4. Docker 20.10.2 ( $ apt install -y docker.io)
    5. Docker Compose 1.25.0 ( $ apt install -y docker-compose)

Mission:
    1. Deploy Jitsi on Docker
    2. Monitoring Jitsi
```

## 1. Deploy Jitsi on Docker
    Jitsi docker source : https://github.com/jitsi/docker-jitsi-meet/releases/latest
```
Installation:
Goes to Directory 1.Jitsi-Docker5765-1
Edit an SSL options,etc:
$ cd 1.Jitsi-Docker5765-1
$ cp env.txt .env
$ nano .env

After .env has been configured:
$ docker-compose up -d

If you failed install jitsi from my repo, you can install from official repo jitsi https://github.com/jitsi/docker-jitsi-meet/releases/latest 

Make sure jitsi componenct has been deployed:
$ docker ps
jitsi/jvb , jitsi/web , jitsi/prosody , jitsi/jicofo

if all have been fulfilled, jitsi can be accessed https://localhost:80

if you want to modify the frontend of your jitsi, you can go to the "3.TemplateJitsiFrontEnd" folder and edit the file:
$ nano app.bundle.min.js
use ctrl+f to search for the keyword "CHANGE HEADER TITTLE HERE" to change the header title and search for "HEADER SUBTITTLE HERE" to change the header subtitle.
save, exit.

if you have, you can move edited file into the jitsi / web container by:
$ docker cp app.bundle.min.js container-jitsi-web-id:/usr/share/jitsi-meet/libs

last, if you want to reinstall jitsi, you can delete the config folder first, then start with docker-compose up again.
$ sudo rm -rf /root/.jitsi-meet-cfg/
$ docker container prune (optional)
$ docker network prune (optional)
$ docker-compose up -d
```

## 2. Monitoring Jitsi
```
Monitoring Jitsi using "telegraph" as the server metric and "influxdb" as the database & "Grafana" as the Dashboard GUI Monitoring.
```

```
Make sure the jitsi container is running.
Installation:
Goes to folder "2.Jitsi-Monitoring"
Edit some conf:
a. telegraf.conf
$ cd conf/
$ nano telegraf.conf/
EDIT:
[[inputs.http_response]]
  name_override = "jvb-health"
  ## List of urls to query.
  urls = ["http://your.domain:8080/about/health"]
  
[[inputs.http]]
  name_override = "jvb-stats"

  ## URL of each server in the service's cluster
  urls = [
    "http://your.domain:8080/colibri/stats",
  ]

save,exit.

b. grafana-jvb.json
$ cd fixtures/
$ nano grafana-jvb.json
modify all domains into your jitsi web domain.

save,exit.

$ docker-compose up -d or you can $ ./up.sh
$ docker ps (make sure container monitoring Jitsi has been deployed)

After all the containers are running go to the JVB container to slightly change the configuration of the JVB:
$ docker ps (copy ID Container for jitsi/jvb)
$ docker exec -it ID-CONTAINER-JVB bash
$ cd /etc/jitsi/videobridge/
$ nano config
change to this :
JVB_OPTS="--apis=rest"

$ nano sip-communicator.properties
add this:
org.jitsi.videobridge.rest.private.jetty.port=8080
org.jitsi.videobridge.ENABLE_STATISTICS=true
org.jitsi.videobridge.STATISTICS_TRANSPORT=muc,colibri

save, exit.

check access colibri to make sure colibri is work:
http://localhost:8080/colibri/stats (if the server metric appears, continue to the next step).

Monitoring on Grafana, access on:
http://localhost:3000

Little config on Grafana:
1. Login with, admin / pass
2. Create a data source "InfluxDB"
fill URL with the name of the service influxdb container, which is http://influxdb: 8086 and server to access type.
for DB:
Fill in the database name given by the telegraph, which can be checked in conf > telegraf.conf
$ nano conf/telegraf.conf 
according to each configuration by looking at line 82. (default: jvb-stats).

And the username for the password can be seen in the .env file
$ nano .env
copy and paste the username and password.

Database summary:
Database : jvb-stats
Username : tiguser
Password : tigpass

Save&Test. if succeess, continue to next step.

3. Make a Dashboard:
import dashboard from "grafana-jvb.json" file. (make sure URL on grafana-jvb.json has been edited to your URL)
Load & Create.

Happy monitoring.
```
## ADDITIONAL NOTES
```
In some cases monitoring still doesn't work due to isolated docker networks, it's a good idea to bridge both parties by:

$ docker network ls (search the name of the jitsi network, usually "dockerjitsi_meet.jitsi" (can be different in each place), if it is visible, copy that name.)
$ docker ps (look at the docker monitoring container, it is a container that we will connect to the jitsi network so that it can access the server metrics.)

$ docker network connect dockerjitsi_meet.jitsi jitsi-monitoring_influxdb_1
$ docker network connect dockerjitsi_meet.jitsi jitsi-monitoring_grafana_1
$ docker network connect dockerjitsi_meet.jitsi jitsi-monitoring_telegraf_1

keep in mind that the container names and network names are adjusted accordingly.
```
