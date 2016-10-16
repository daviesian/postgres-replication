# PostgreSQL Replication with Docker

## Configuration

* Make sure nothing exists already: 

  `docker-compose down -v`
* Create a blank DB in volume `data-1`, then start service `pg1` using it:

  `docker-compose run --rm pg1-create-primary && docker-compose up pg1`
* From another server, back up the `data-1` db into `data-2`, then run `pg2` as a hot-standby:

  `docker-compose run --rm pg2-create-standby && docker-compose up pg2`
  
* Check that it's working:
  ```
  # Create table on pg1
  docker-compose exec pg1 gosu postgres psql user -c 'create table foo (x text)'
  
  # List tables on pg2
  docker-compose exec pg2 gosu postgres psql user -c '\d'
  
  # pg2 is read-only - this should fail:
  docker-compose exec pg2 gosu postgres psql user -c "insert into foo values ('x')"
  ```
  
## Failover

* On first server, kill `pg1`
* Promote `pg2` to primary: 

  `docker-compose exec pg2 touch /recover.trigger`
* Check that it's working:
  ```
  # Should now be able to write to pg2
  docker-compose exec pg2 gosu postgres psql user -c "insert into foo values ('x')"
  ```

## Failback

* Bring `pg1` back up, this time as a hot standby of `pg2`:

  `docker-compose run --rm pg1-create-standby && docker-compose up pg1`
* Check that it's mirroring as above.
* Kill `pg2`
* Promote `pg1` back to primary: 

  `docker-compose exec pg1 touch /recover.trigger`
