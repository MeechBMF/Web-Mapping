FROM postgis/postgis:16-3.4
# Ensures PostGIS is enabled in the app database on first start
COPY init-postgis.sql /docker-entrypoint-initdb.d/10-init-postgis.sql