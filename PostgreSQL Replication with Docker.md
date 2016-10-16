Compose file:

```yaml
version: '2'
services:
  pg1:
    container_name: pg1
    image: postgres:9.6.0
    volumes:
      - ./pg1-data:/var/lib/postgresql/data
      - ./pg_hba.conf:/var/lib/postgresql/data/pg_hba.conf
      - ./postgresql.conf:/var/lib/postgresql/data/postgresql.conf
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: passw0rd
  pg2:
    container_name: pg2
    image: postgres:9.6.0
    volumes:
      - ./pg2-data:/var/lib/postgresql/data
      - ./pg_hba.conf:/var/lib/postgresql/data/pg_hba.conf
      - ./postgresql.conf:/var/lib/postgresql/data/postgresql.conf
      - ./recovery.conf.pg2_standby:/var/lib/postgresql/data/recovery.conf
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: passw0rd
```

* Make sure nothing exists already: `docker-compose down -v`
* Comment out all but first volume in both pg1 and pg2
* Create DB data directory for db1 (not necessary if you have a DB already): `docker-compose run --rm pg1`
* Start pg1: `docker-compose up -d pg1`
* Backup DB to pg2: `docker-compose run --rm pg2 gosu postgres pg_basebackup -h pg1 -D /var/lib/postgresql/data -U user -v -P`
* Start pg2: `docker-compose up -d pg2`

`pg2` should now be hot-standby for pg1.
* Verify:
  ```
  # Create table on pg1
  docker exec pg1 gosu postgres psql user -c 'create table foop (x text)'
  
  # List tables on pg2
  docker exec pg2 gosu postgres psql user -c '\d'
  ```
  
## Creation
