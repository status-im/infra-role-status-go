# Description

This role configures a [`status-go`](https://github.com/status-im/status-go) __history node__ for archiving whisper envelopes.

# Ports

* `30504` - DevP2P port for peer communicaiton (__public__)
* `443` - Alternative peer port for avoiding firewalls (__public__)
* `8545` - HTTP JSON RPC port for administration (__private__)
* `9305` - Prometheus Metrics port (__private__)
* `52525` - Go Performance Profiling port (__private__)

# Configuration

The most important settings would be:
```yaml
mailsrv_node_cont_tag: 'deploy-test'
mailsrv_waku_enabled: true
mailsrv_log_level: 'DEBUG'
```

# Usage

The containers are created using [Docker Compose](https://docs.docker.com/compose/) and consist of the `status-go` node and PostgreSQL database:
```
admin@mail-01.do-ams3.eth.test:~ % docker ps 
CONTAINER ID        NAMES               IMAGE                              CREATED             STATUS
dca9b2a708c2        statusd-mail-node   statusteam/status-go:deploy-test   2 minutes ago       Up 4 seconds
bfed6063abe9        statusd-mail-db     postgres:9.6-alpine                2 minutes ago       Up 2 minutes
```
You can manage the containers using `docker-compose`. To re-create them use:
```
admin@mail-01.do-ams3.eth.test:/docker/statusd-mailsrv % docker-compose --compatibility up --force-recreate -d
Recreating statusd-mail-db ... done
Recreating statusd-mail-node ... done
```

# Backups

Backups of the PostgreSQL DB are done using [systemd timers](https://www.freedesktop.org/software/systemd/man/systemd.timer.html) and can be viewed using `systemctl`:
```
admin@mail-01.do-ams3.eth.test:~ % sudo systemctl -a list-timers 'dump-statusd-mail*'   
NEXT                         LEFT     LAST PASSED UNIT                       ACTIVATES
Wed 2020-04-01 00:00:00 UTC  10h left n/a  n/a    dump-statusd-mail-db.timer dump-statusd-mail-db.service
```
And ran manually:
```
admin@mail-01.do-ams3.eth.test:~ % sudo systemctl start dump-statusd-mail-db
```
And check logs:
```
admin@mail-01.do-ams3.eth.test:~ % sudo journalctl -o cat -u dump-statusd-mail-db
Starting Dumping mailserver PostgreSQL database...
Created: /var/tmp/backups/mailsrv/statusd_mail_db_dump_20200331130625.sql
Started Dumping mailserver PostgreSQL database.
```
