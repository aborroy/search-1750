# Sample configuration for SEARCH-1750

This project includes base configuration to reproduce [SEARCH-1750](https://issues.alfresco.com/jira/browse/SEARCH-1750) issue.

This configuration includes two Shard Instances with EXPLICIT_ID sharding method and an additional replica of Shard Instance 0 that starts after the system has been working for a while:

* http://localhost:8083/solr - Shard Instance 0
* http://localhost:8084/solr - Shard Instance 0 (replica)
* http://localhost:8085/solr - Shard Instance 1

The Sharding Method property is named `shard:shardId`, belonging to aspect `shard:sharding`, and it's deployed by default with ACS Docker Image.

## Usage

Start the environment without Shard Instance 0 replica.

```
$ docker-compose up --build --force-recreate --scale solr6replica=0

$ docker ps --format 'table {{.Names}}\t{{.Image}}'
NAMES                                 IMAGE
search-1750_alfresco-pdf-renderer_1   alfresco/alfresco-pdf-renderer:2.1.0-EA4
search-1750_transform-router_1        quay.io/alfresco/alfresco-transform-router:1.1.0-EA2
search-1750_libreoffice_1             alfresco/alfresco-libreoffice:2.1.0-EA4
search-1750_tika_1                    alfresco/alfresco-tika:2.1.0-EA4
search-1750_transform-misc_1          alfresco/alfresco-transform-misc:2.1.0-EA4
search-1750_imagemagick_1             alfresco/alfresco-imagemagick:2.1.0-EA4
search-1750_proxy_1                   alfresco/alfresco-acs-nginx:3.0.1
search-1750_alfresco_1                search-1750_alfresco
search-1750_digital-workspace_1       quay.io/alfresco/alfresco-digital-workspace:1.1.0
search-1750_share_1                   alfresco/alfresco-share:6.1.0-RC3
search-1750_solr6secondary_1          search-1750_solr6secondary
search-1750_activemq_1                alfresco/alfresco-activemq:5.15.8
search-1750_postgres_1                postgres:10.9
search-1750_solr6_1                   search-1750_solr6
search-1750_shared-file-store_1       alfresco/alfresco-shared-file-store:0.5.3
```

Create a folder for indexes in Shard Instance 0.

```
ShardId
Rule applied to subfolders

When:
Items are created or enter this folder

If all criteria are met:
All Items

Perform Action:
Set property value > Property:shard:shardId Value:0
```

Create a folder for indexes in Shard Instance 1.

```
ShardId
Rule applied to subfolders

When:
Items are created or enter this folder

If all criteria are met:
All Items

Perform Action:
Set property value > Property:shard:shardId Value:1
```

Add some contents and create some folders inside these folders using Share web app (http://localhost:8080/share)

Check `numFound` property for Shard Instance 0 using following URL:

```
http://localhost:8083/solr/alfresco/select?fl=[cached]cm_name,[cached]PATH,[cached]DBID&indent=on&q=cm:name:*&sort=DBID%20desc&wt=json
```

Check `numFound` property for Shard Instance 1 using following URL:

```
http://localhost:8085/solr/alfresco/select?fl=[cached]cm_name,[cached]PATH,[cached]DBID&indent=on&q=cm:name:*&sort=DBID%20desc&wt=json
```

Start Shard Instance 0 replica.

```
$ docker-compose up --build --force-recreate solr6replica

$ docker ps --format 'table {{.Names}}\t{{.Image}}'
NAMES                                 IMAGE
search-1750_solr6replica_1            search-1750_solr6replica
search-1750_alfresco-pdf-renderer_1   alfresco/alfresco-pdf-renderer:2.1.0-EA4
search-1750_transform-router_1        quay.io/alfresco/alfresco-transform-router:1.1.0-EA2
search-1750_libreoffice_1             alfresco/alfresco-libreoffice:2.1.0-EA4
search-1750_tika_1                    alfresco/alfresco-tika:2.1.0-EA4
search-1750_transform-misc_1          alfresco/alfresco-transform-misc:2.1.0-EA4
search-1750_imagemagick_1             alfresco/alfresco-imagemagick:2.1.0-EA4
search-1750_proxy_1                   alfresco/alfresco-acs-nginx:3.0.1
search-1750_alfresco_1                search-1750_alfresco
search-1750_digital-workspace_1       quay.io/alfresco/alfresco-digital-workspace:1.1.0
search-1750_share_1                   alfresco/alfresco-share:6.1.0-RC3
search-1750_solr6secondary_1          search-1750_solr6secondary
search-1750_activemq_1                alfresco/alfresco-activemq:5.15.8
search-1750_postgres_1                postgres:10.9
search-1750_solr6_1                   search-1750_solr6
search-1750_shared-file-store_1       alfresco/alfresco-shared-file-store:0.5.3
```

Once everything is indexed in this Shard, perform the same operation and check `numFound` property using following URL:

```
http://localhost:8084/solr/alfresco/select?fl=[cached]cm_name,[cached]PATH,[cached]DBID&indent=on&q=cm:name:*&sort=DBID%20desc&wt=json
```

The result must be the same found for Shard Instance 0 (http://localhost:8083/solr)
