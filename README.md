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
status_go_node_cont_tag: 'deploy-test'
status_go_waku_v1_enabled: true
status_go_waku_v2_enabled: false
status_go_history_enabled: false
status_go_log_level: 'DEBUG'
```
If you want to provide specific node key you can use:
```
status_go_node_key: 'abc321...'
```

# Usage

The containers are created using [Docker Compose](https://docs.docker.com/compose/) and consist of the `status-go` node and PostgreSQL database:
```
admin@mail-01.do-ams3.eth.test:~ % docker ps 
CONTAINER ID        NAMES               IMAGE                              CREATED             STATUS
dca9b2a708c2        status-go-node   statusteam/status-go:deploy-test   2 minutes ago       Up 4 seconds
bfed6063abe9        status-go-db     postgres:9.6-alpine                2 minutes ago       Up 2 minutes
```
You can manage the containers using `docker compose`. To re-create them use:
```
admin@mail-01.do-ams3.eth.test:/docker/status-go % docker compose --compatibility up --force-recreate -d
Recreating status-go-db ... done
Recreating status-go-node ... done
```

# Backups

Backups of the PostgreSQL DB are done using [systemd timers](https://www.freedesktop.org/software/systemd/man/systemd.timer.html) and can be viewed using `systemctl`:
```
admin@mail-01.do-ams3.eth.test:~ % sudo systemctl -a list-timers 'dump-status-go*'   
NEXT                         LEFT     LAST PASSED UNIT                       ACTIVATES
Wed 2020-04-01 00:00:00 UTC  10h left n/a  n/a    dump-status-go-db.timer dump-status-go-db.service
```
And ran manually:
```
admin@mail-01.do-ams3.eth.test:~ % sudo systemctl start dump-status-go-db
```
And check logs:
```
admin@mail-01.do-ams3.eth.test:~ % sudo journalctl -o cat -u dump-status-go-db
Starting Dumping mailserver PostgreSQL database...
Created: /var/tmp/backups/mailsrv/status_go_db_dump_20200331130625.sql
Started Dumping mailserver PostgreSQL database.
```
