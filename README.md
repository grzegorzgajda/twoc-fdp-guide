# twoc-fdp-guide
Guide for deploying a FAIR Data Point for TWOC.

## Docker
```sh
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
sudo usermod -aG docker $USER
newgrp docker
```

https://docs.docker.com/compose/install/#install-compose-on-linux-systems
```sh
sudo curl -L "https://github.com/docker/compose/releases/download/1.28.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

## GraphDB
https://www.ontotext.com/products/graphdb/graphdb-free/

from the email -> "Download as a stand-alone server"

graphdb-free-9.6.0-dist.zip

```sh
sudo mkdir /opt/graphdb
sudo chown $USER:$USER /opt/graphdb
sudo apt-get install -y -qq git-core
git clone https://github.com/Ontotext-AD/graphdb-docker.git /opt/graphdb
mv graphdb-free-9.6.0-dist.zip /opt/graphdb/free-edition
cd /opt/graphdb
docker-compose build --build-arg version=9.6.0

docker network create graphdb_net

truncate -s -1 docker-compose.yml
cat <<EOL >> docker-compose.yml
    networks:
      - "graphdb_net"

networks:
  graphdb_net:
    external: true
EOL

docker-compose up -d
```

### first use
- Open http://twoc.example.com:7200
- Create `fdp` repository
- Setup -> My Settings
  - Enter and confirm admin password
- Setup -> Users and Access
  - Enable security

## FAIR Data Point
```sh
sudo mkdir /opt/twoc-deployment
sudo chown $USER:$USER /opt/twoc-deployment
git clone --branch develop https://github.com/TrustedWorldOfCorona/twoc-fdp-guide.git /opt/twoc-deployment
cd /opt/twoc-deployment
docker-compose up -d proxy
sudo apt-get install -y -qq certbot
sudo certbot certonly --webroot -w ./proxy/letsencrypt -d twoc.example.com
docker-compose down
nano proxy/nginx/nginx.conf # and uncomment the include line
nano proxy/nginx/fdp.conf # update domains
docker-compose up -d
```

### certificate renewal hooks
```sh
sudo tee /etc/letsencrypt/renewal-hooks/post/001-restart-docker-proxy.sh <<EOL
#!/bin/sh
docker restart twoc-deployment_proxy_1
EOL
sudo chmod +x /etc/letsencrypt/renewal-hooks/post/001-restart-docker-proxy.sh
```
