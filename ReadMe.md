docker pull zabbix/zabbix-agent:ubuntu-6.4-latest
docker pull zabbix/zabbix-server-mysql:ubuntu-6.4-latest
docker pull zabbix/zabbix-web-nginx-mysql:ubuntu-6.4-latest
docker pull zabbix/zabbix-proxy-mysql:ubuntu-6.4-latest
docker pull zabbix/zabbix-java-gateway:ubuntu-6.4-latest
docker pull zabbix/zabbix-snmptraps:ubuntu-6.4-latest


docker network create --subnet 172.20.0.0/16 --ip-range 172.20.240.0/20 zabbix-net

docker run --name mysql-server -t \
      -e MYSQL_DATABASE="zabbix" \
      -e MYSQL_USER="zabbix" \
      -e MYSQL_PASSWORD="password" \
      -e MYSQL_ROOT_PASSWORD="password" \
      --network=zabbix-net \
      --restart unless-stopped \
      -d mysql:8.0-oracle \
      --character-set-server=utf8 --collation-server=utf8_bin \
      --default-authentication-plugin=mysql_native_password
	  
	  
docker run --name zabbix-java-gateway -t \
      --network=zabbix-net \
      --restart unless-stopped \
      -d zabbix/zabbix-java-gateway:ubuntu-6.4-latest
	  
docker run --name zabbix-snmptraps -t \
      -v /zbx_instance/snmptraps:/var/lib/zabbix/snmptraps:rw \
      -v /var/lib/zabbix/mibs:/usr/share/snmp/mibs:ro \
      --network=zabbix-net \
      -p 162:1162/udp \
      --restart unless-stopped \
      -d zabbix/zabbix-snmptraps:ubuntu-6.4-latest
	  
	  
docker run --name zabbix-server-mysql -t \
      -e DB_SERVER_HOST="mysql-server" \
      -e MYSQL_DATABASE="zabbix" \
      -e MYSQL_USER="zabbix" \
      -e MYSQL_PASSWORD="password" \
      -e MYSQL_ROOT_PASSWORD="password" \
      -e ZBX_JAVAGATEWAY="zabbix-java-gateway" \
      --network=zabbix-net \
      -p 10051:10051 \
	  --volumes-from zabbix-snmptraps \
      --restart unless-stopped \
      -d zabbix/zabbix-server-mysql:ubuntu-6.4-latest
	  
	  
docker run --name zabbix-web-nginx-mysql -t \
      -e ZBX_SERVER_HOST="zabbix-server-mysql" \
      -e DB_SERVER_HOST="mysql-server" \
      -e MYSQL_DATABASE="zabbix" \
      -e MYSQL_USER="zabbix" \
      -e MYSQL_PASSWORD="password" \
      -e MYSQL_ROOT_PASSWORD="password" \
      --network=zabbix-net \
      -p 80:8080 \
      --restart unless-stopped \
      -d zabbix/zabbix-web-nginx-mysql:ubuntu-6.4-latest
	  
apt install mysql-client-core-8.0
apt-get install mysql-server
