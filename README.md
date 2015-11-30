``` console
╺┳┓╻ ╻   ┏━╸╻  ┏━┓┏━┓┏━┓╻┏━╸╻┏━╸┏━┓╺┳╸╻┏━┓┏┓╻┏━┓
 ┃┃┃╻┃╺━╸┃  ┃  ┣━┫┗━┓┗━┓┃┣╸ ┃┃  ┣━┫ ┃ ┃┃ ┃┃┗┫┗━┓
╺┻┛┗┻┛   ┗━╸┗━╸╹ ╹┗━┛┗━┛╹╹  ╹┗━╸╹ ╹ ╹ ╹┗━┛╹ ╹┗━┛
```

Introduction
------------

This is a integration project that packages the PlutoF Classifications module as a docker application.

### The "docker application"

The `docker-compose.yml` text file defines the docker applications composition, currently:

-   db ... a postgres database container
-   es ... an elastic search container
-   web ... a python/django application server

TODO: add reverse web proxy `nginx` with SSL.

### Dockerfile

This defines the the `web` container, which can be built with:

``` bash
docker-compose build web

# Network issues?
# you may need to ensure your host provides DNS to containers
# ... if on linux, and your containers cannot dl content, ...
# see /etc/default/docker amend dns settings with info from 
# "nm-tool | grep DNS"
```

### Search with `elasticsearch`

Two directories are used by Elastic Search for config and persistance, but we make no specific customizations:

-   es-conf
-   es-data

### Database

The `pg-init` directory contains a .sql file which creates a user and a database, ready for use by the Python/Django Classifications application.

### The PlutoF Classifications Module setup

Customizations for the module are in these dirs:

-   plutof-taxonomy-module ... this is the github repo
-   plutof-conf ... this contains various customizations to the PlutoF Classifications module
-   plutof-data ... these are tools for data import and export

Scripts
-------

There are three scripts for bootstrapping the system:

-   up.sh ... NOTE! use this for starting up the first time
-   pre\_up.sh ... called by up.sh to the latest module
-   post\_up.sh ... called by up.sh to install module db schema and run tests

After starting the system, use standard docker commands such as:

``` bash
docker-compose ps
docker-compose stop
docker-compose rm -v -f
docker-compose up -d
docker-compose logs
docker-compose start
```

Loading content
===============

In the `plutof-data` directory there are tools for loading data:

-   `dyntaxa.py` downloads data from <https://taxon.artdatabankensoa.se/TaxonService.svc?wsdl>
-   `dyntaxa-credentials.cfg` contains credentials for using the web service @ artdatabanken
-   `animalia.py` attempts to list all nearest children at phylum rank below Animalia
-   `animalia_xml2csv.py` converts XML generated by the animalia script to CSV
-   `xml_batch_upload.py` uploads data to PlutoF Classifications module using the REST API

The above tools are used in two steps when loading data:

-   The `dl.sh` attempts to automate downloading of DynTaxa data. It tends to choke on some large trees. That is why the `animalia.py` script was created.
-   The `ul.sh` takes .xml payloads from and authenticates and uploads these to the REST API.

Questions / issues / discussions
================================

-   How to map use nice persistent identifiers in the module?
-   Intermediary .csv format or dividing the upload tool into two steps...
-   Nice if the batch upload be made from .csv ...
-   ... in order to avoid current specifics related to DynTaxa xml and provide a general upload tool?
-   Can the batch upload take more options?
-   For example destination (<http://localhost:7000>)
-   Also the client identifier and client secret, read from a .cfg?
-   How to programmatically add a "client" to use for importing data?
-   Can we use `plutof-conf/oauth2_client.json`?
-   How to avoid any manual steps currently required at <http://localhost:7000/admin>?

License
-------

Affero GPLv3 license
