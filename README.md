# powerdns

öncelikle docker kurulu bir host üzerinde powerdns in kullanacağı bir db ayağa kaldıracağız.

```
docker run -d -e MYSQL_ROOT_PASSWORD=pdns-secret --name mariadb mariadb
```

ardından powerdns ayağa kaldıracağız.

````
docker run -d -p 53:53 -p 53:53/udp --name pdns \
  --hostname ns1.devopsdude.info --link mariadb \
  -e PDNS_gmysql_password=pdns-secret \
  -e PDNS_master=yes \
  -e PDNS_api=yes \
  -e PDNS_api_key=secret \
  -e PDNS_webserver=yes \
  -e PDNS_webserver_address=0.0.0.0 \
  -e PDNS_webserver_password=secret2 \
  -e PDNS_version_string=anonymous \
  -e PDNS_default_ttl=1500 \
  pschiffe/pdns-mysql
  ````

admin arayüzü için bir backend bir de frontend yaratacağız.

backend:

````
docker run -d --name pdns-admin-uwsgi \
  --link mariadb --link pdns \
  -v pdns-admin-upload:/opt/powerdns-admin/upload \
  pschiffe/pdns-admin-uwsgi
  ````

frontend:

````
docker run -d -p 80:80 --name pdns-admin-static \
  --link pdns-admin-uwsgi \
  pschiffe/pdns-admin-static
  ````

