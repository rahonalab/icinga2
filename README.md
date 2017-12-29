# icinga2-docker

This repository contains a (quasi-)modular image of the Icinga2 monitor systems, orchestrated by docker-compose; to ensures portability through different versions of containers, it uses permanent volumes for storage of config files and data.

It is slightly based on the original Dockerfile by [https://github.com/jjethwa/icinga2] (Jordan Jethwa's icinga2 docker image), which is also available as a dockerhub-repository is located at [https://hub.docker.com/r/jordan/icinga2/](https://hub.docker.com/r/jordan/icinga2/).

## Image details

1. Features four containers:
   - core: the icinga2 system, plus the ssmtp facility
   - web: icingaweb2, the web-based, eye-candy gui, plus pnp4nagios
   - sql: library/mariadb (no modification, no dockerfile, just as-is)
   - snmptrap: an snmptrap system based on snmptt, snmptrapd
   
1. Based on debian:jessie-slim (core, snmptrap) and debian:stretch-slim (web)

1. Key-Features:
   - icinga2
   - auto-setup as a satellite or a master, you choose!
   - icingacli
   - icingaweb2
   - icingaweb2-pnp4nagios module
   - ssmtp
   - MariaDB
   - Supervisor
   - Apache2
   - SSL Support
   - pnp4nagios
   - a bunch of special plugins for monitoring ups, printer and temp sensor (via SNMP)

## Usage

Images are not (yet) available on docker hub, sorry!

However, just clone this repository, set variables in secrets.env and .env and build the image with:

    docker-compose build

then start the four containers:

    docker-compose up

et voilà, you are set! You will find:

  - icinga2 running on port 5665;
  - icingaweb running on port 80;
  - snmptrap running on port 162

The system is meant to be run in a master-satellite setup, as described [here](https://www.icinga.com/docs/icinga2/latest/doc/06-distributed-monitoring/); set the ${TYPE} variable in secrets.env to your need. If you are not interested in master-satellite setup, just set the variable to 'master'.

To remove one container (but not conf and data stored, see below!) do:

   docker-compose rm core|web|snmptrap|sql

or:

   docker-compose rm

to remove all the containers.

## Volume Reference

Directives in docker-compose.yaml create a series of directories in the ${FIRSTNAME}/ directory located into icinga2-docker directory; this ensures the portability of configuration and data through different versions of containers.

In order to work in a full clean environment, just remove the ${FIRSTNAME}/ (or just parts of it) before running new containers.


| Host | Container:directory | Description & Usage |
| ------ | ----- | ------------------- |
| ./${FIRSTNAME}-container/etc/icinga2 | core:/etc/icinga2 | Icinga2 configuration dir |
| ./${FIRSTNAME}-container/lib/icinga2 | core:/var/lib/icinga2 | Library dir for icinga2. You find certificate files here. |
| ./${FIRSTNAME}-container/nagios-plugins | core:/usr/lib/nagios/plugins | Plugins dir for Icinga2 |
| ./${FIRSTNAME}-container/cache/icinga2 | core:/var/cache/icinga2 | Cache dir for Icinga2. Just for debug |
| ./${FIRSTNAME}-container/log/icinga2 | core:/var/log/icinga2 | Log dir for Icinga2 |
| ./${FIRSTNAME}-container/run/icinga2 | core:/var/run/icinga2 | Run dir for icinga2. Send external command on icinga2.cmd, used by web |
| ./${FIRSTNAME}-container/spool/icinga2 | core:/var/spool/icinga2 | Spool dir for icinga2. Contains perf data used by pnp4nagios  |
| ./${FIRSTNAME}-container/etc/icingaweb2 | web:/etc/icingaweb2 | Icingaweb2 configuration dir |

(**more to describe**)

## Update an existing system
If you already have a working Icinga2 system, just copy your config (/etc/icinga2) and certificate (/var/lib/icinga2/certs or /etc/icinga2/pki for icinga < 2.8) files in the directories listed above. The setup will automatically detect the presence of certificate files in /var/lib/icinga2/certs and will skip the configuration process.

## Icinga Web 2

Icinga Web 2 can be accessed at [http://localhost/icingaweb2](http://localhost/icingaweb2) with the credentials set in secrets.env

# Sending Notification Mails

The core container has `ssmtp` installed, which forwards mails to a preconfigured static server.

You have to create the files `ssmtp.conf` for general configuration and `revaliases` (mapping from local Unix-user to mail-address).

```
# ssmtp.conf
root=<E-Mail address to use on>
mailhub=smtp.<YOUR_MAILBOX>:587
UseSTARTTLS=YES
AuthUser=<Username for authentication (mostly the complete e-Mail-address)>
AuthPass=<YOUR_PASSWORD>
FromLineOverride=NO
```
**But be careful, ssmtp is not able to process special chars within the password correctly!**

`revaliases` follows the format: `Unix-user:e-Mail-address:server`.
Therefore the e-Mail-address has to match the `root`'s value in `ssmtp.conf`
Also server has to match mailhub from `ssmtp.conf` **but without the port**.

```
# revaliases
root:<VALUE_FROM_ROOT>:smtp.<YOUR_MAILBOX>
nagios:<VALUE_FROM_ROOT>:smtp.<YOUR_MAILBOX>
www-data:<VALUE_FROM_ROOT>:smtp.<YOUR_MAILBOX>
```

These files have to get mounted into the container. Add these flags to your `docker run`-command:
```
-v $(pwd)/revaliases:/etc/ssmtp/revaliases:ro
-v $(pwd)/ssmtp.conf:/etc/ssmtp/ssmtp.conf:ro
```

If you want to change the display-name of sender-address, you have to define the variable `ICINGA2_USER_FULLNAME`.

If this does not work, please ask your provider for the correct mail-settings or consider the [ssmtp.conf(5)-manpage](https://linux.die.net/man/5/ssmtp.conf) or Section ["Reverse Aliases" on ssmtp(8)](https://linux.die.net/man/8/ssmtp).
Also you can debug your config, by executing inside your container `ssmtp -v $address` and pressing 2x Enter.
It will send an e-Mail to `$address` and give verbose log and all error-messages.

# Adding own modules

To use your own modules, you're able to install these into `enabledModules`-folder of your `/etc/icingaweb2` volume.

## Environment variables Reference

TODO
