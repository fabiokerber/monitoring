# ELASTIC

**Stack:**<br>
- ElasticSearch: 7.17.3<br>
- Kibana: 7.16.1<br>

## API - Dev Tools [🔗cURL](https://curl.se/docs/manpage.html)<br>

**User**<br>
```yml
GET _security/user
GET _security/user/<user>
```

**Role**<br>
```yml
GET _security/role
GET _security/role/<role>
```

**Index**<br>
```yml
GET <index>
GET _cat/indices/<index>*?v=true&s=index
GET _cat/indices/<index>*?v=true&s=index
GET <index>/_settings?filter_path=*.settings.index.lifecycle
GET <index>*/_settings?filter_path=*.settings.index.lifecycle
GET _cat/indices/<index>*?v=true&s=index&h=index,store.size&bytes=gb
GET _cat/indices/<index>*-fe-*?v=true&s=docs.count:desc
GET _cat/indices/<index>*-db-*?v=true&s=docs.count:desc

GET _component_template
GET _component_template/<component_template>

GET _index_template
GET _index_template/<index_template>
```

**Force Rollover**<br>
```yml
POST <alias>/_rollover/
```

**Aliases**<br>
```yml
GET _aliases
GET _aliases/<alias>
```

**ILM**<br>
```yml
GET _ilm/policy
GET _ilm/policy/<policy>
GET <index>/_ilm/explain?human
GET <index>*/_ilm/explain?human
GET /_ilm/status

*CHECK ILM EFFECTIVENESS*
GET *filebeat*?human&filter_path=*.settings.index.creation_date_string,*.settings.index.lifecycle.name

GET _cat/indices/*filebeat*?v=true&h=health,status,index,creation.date.string,docs.count,store.size,pri.store.size&s=creation.date.string:asc
GET _ilm/policy/?filter_path=*-days-policy.policy.phases,*-days-policy.in_use_by.indices
```

**Shards**<br>
```yml
GET _cat/shards/?v=true
GET _cat/shards/<index>?v=true&s=prirep
GET _cat/shards/<index>*?v=true&s=prirep
```

**Elastic System**<br>
```yml
*ILM*
GET _cluster/settings?include_defaults=true&filter_path=*.indices.lifecycle*,*.xpack.ilm*

*ROLLOVER*
POST <alias>/_rollover/

*MASTER NODE*
GET _cat/master?v&pretty

*ALL NODES*
GET _cat/nodes?v&pretty

*TASKS*
GET _cat/tasks?v&pretty

*CPU USAGE*
GET _cat/nodes?v=true&s=cpu:desc

*DISK USAGE*
GET _cat/allocation?v&pretty

*HEALTH*
GET _cat/health?v&pretty

*HELP*
GET _cat/indices/?help
```

```yml
> Vagrantfile
  elk.vm.provision 'shell', inline: 'dockerd --max-concurrent-downloads 2 &>/dev/null' ("Weak Network")

- TOKEN -
$ vagrant up elk
$ vagrant ssh elk
$ sudo docker exec -it elasticsearch /bin/bash
$ /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token --scope kibana

> http://LOG_IP:3000
  admin
  admin

> http://LOG_IP:5601

> http://LOG_IP:9200
```

# TEMPLATE

- Space: `new_space`
- Background color: `background_color`
- Index: `new_index`
- Index component template: `default-replica-component-template`
- Index component template: `default-mapping-component-template`
- Index template: `new_index_template`
- Role: `new_role`
- Alias: `new_alias`
- ILM Policy: `15-days-policy`
- User: `new_user`
- User full name: `full_name`
- User email: `email`<br>
**Space = Role**

## Spaces ▶︎ `Management > Spaces`<br>

**Create space `new_space` [🔗Spaces](https://www.elastic.co/guide/en/kibana/master/xpack-spaces.html)**<br>
- **+ Create space**
  - **Name:** `new_space`
  - **Background color:** `background_color`
  - **Analytics**
    - Discover
    - Dashboard
    - Canvas
    - Maps
    - Machine Learning
    - Visualize Library
  - **Observability**
    - Logs
    - Metrics
  - **Management**
    - Advanced Settings
    - Index Pattern Management
    - Saved Objects Management

## User ▶︎ `Dev Tools`<br>

**Create `new_user` with default role `run_as` [🔗Configure security in Kibana API](https://www.elastic.co/guide/en/kibana/7.17/using-kibana-with-security.html)**<br>
```json
PUT _security/user/<new_user>
{
  "password" : "${openssl rand -hex 32}",
  "roles" : ["run_as"],
  "full_name" : "<full_name>",
  "email" : "<email>"
}
```

**Add `new_user` to role `run_as` [🔗Create or update roles API](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/security-api-put-role.html)**<br>
```diff
- Run GET to check if there are other users in the role, before running PUT
```
```json
PUT _security/role/run_as
{
  "cluster" : [ ],
  "indices" : [ ],
  "applications" : [ ],
  "run_as": [
    "<new_user>"
  ],
  "metadata" : {
    "version" : 1
  }
}
```

## Role ▶︎ `Dev Tools`<br>

**Create role `new_role`, including `new_alias`, `new_space` and `new_user` [🔗Create or update roles API](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/security-api-put-role.html)**<br>
```diff
- Run GET to check if already exist this role, before running PUT
```
```json
PUT _security/role/<new_role>
{
  "cluster" : [ ],
  "indices" : [
    {
      "names" : [
        "<new_alias>*"
      ],
      "privileges" : [
        "all"
      ],
      "allow_restricted_indices" : false
    }
  ],
  "applications" : [
    {
      "application" : "kibana-.kibana",
      "privileges" : [
          "feature_discover.all",
          "feature_dashboard.all",
          "feature_canvas.all",
          "feature_maps.all",
          "feature_ml.all",
          "feature_visualize.all",
          "feature_logs.all",
          "feature_indexPatterns.all",
          "feature_infrastructure.all",
          "feature_savedObjectsManagement.all"
      ],
      "resources" : [
        "space:<new_space>"
      ]
    }
  ],
  "run_as" : [
    "<new_user>"
  ],
  "metadata" : {
    "version" : 1
  },
  "transient_metadata" : {
    "enabled" : true
    }
}
```

**Set `new_role` to `new_user` [🔗Configure security in Kibana API](https://www.elastic.co/guide/en/kibana/7.17/using-kibana-with-security.html)**<br>
```json
PUT _security/user/<new_user>
{
  "roles" : ["run_as","<new_role>"],
  "full_name" : "<full_name>",
  "email" : "<email>"
}
```

## ILM ▶︎ `Dev Tools`<br>

**Create `15-days-policy` [🔗Create or update lifecycle policy API](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/ilm-put-lifecycle.html)**<br>
```json
PUT _ilm/policy/15-days-policy
{
  "policy" : {
    "phases" : {
      "hot" : {
        "min_age" : "0ms",
        "actions" : {
          "rollover" : {
            "max_primary_shard_size" : "25gb",
            "max_age" : "7d",
            "max_docs" : 40000000
          }
        }
      },
      "warm" : {
        "min_age" : "12h",
        "actions" : {
          "allocate" : {
            "number_of_replicas" : 0,
            "include" : { },
            "exclude" : { },
            "require" : { }
          }
        }
      },
      "cold" : {
        "min_age" : "1d",
        "actions" : {
          "readonly" : { }
          }
        }
      },
      "delete" : {
        "min_age" : "7d",
        "actions" : {
          "delete" : {
            "delete_searchable_snapshot" : true
        }
      }
    }
  }
}
```

**Create `180-days-policy` [🔗Create or update lifecycle policy API](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/ilm-put-lifecycle.html)**<br>
```json
PUT _ilm/policy/180-days-policy
{
  "policy" : {
    "phases" : {
      "hot" : {
        "min_age" : "0ms",
        "actions" : {
          "rollover" : {
            "max_primary_shard_size" : "70gb",
            "max_age" : "90d",
            "max_docs" : 200000000
          }
        }
      },
      "warm" : {
        "min_age" : "1d",
        "actions" : {
          "allocate" : {
            "number_of_replicas" : 0,
            "include" : { },
            "exclude" : { },
            "require" : { }
          }
        }
      },
      "cold" : {
        "min_age" : "2d",
        "actions" : {
          "readonly" : { }
        }
      },
      "delete" : {
        "min_age" : "90d",
        "actions" : {
          "delete" : {
            "delete_searchable_snapshot" : true
          }
        }
      }
    }
  }
}
```

## Index ▶︎ `Dev Tools`<br>

**Create `default-replica-component-template` and `default-mapping-component-template` [🔗Create or update component template API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-component-template.html)**<br>
```json
PUT _component_template/default-replica-component-template
{
  "template": {
    "settings": {
      "number_of_shards": 3,
      "number_of_replicas": 1
    }
  }
}

PUT _component_template/default-mapping-component-template
{
  "template": {
    "mappings": {
      "properties": {
        "created_at": {
          "type": "date",
          "format": "EEE dd MMM yyyy HH:mm:ss Z"
        }
      }
    }
  }
}
```
```diff
+ **Mapping:** Mapped fields will be affected only next index creation
```

**Create `new_index_template` [🔗Create or update index template API](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/indices-templates-v1.html)**<br>
```json
PUT _index_template/<new_index_template>
{
  "template": {
    "settings": {
      "index.lifecycle.name": "15-days-policy",
      "index.lifecycle.rollover_alias": "<new_alias>"
    }
  },
  "index_patterns": [
    "<new_alias>-*"
  ],
  "composed_of": [
    "default-mapping-component-template",
    "default-replica-component-template"
  ]
}
```

**Create `new_index` [🔗Add an alias at index creation](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/aliases.html#add-alias-at-creation)**<br>
```json
PUT <new_alias>-000001
{
  "aliases": {
    "<new_alias>": {
      "is_write_index": true
    }
  }
}
```
```diff
+ **aliases:** "<new_alias1>","<new_alias2>"* > more than one alias to same index
```

## Index Pattern ▶︎ `Management > Stack Management > Kibana > Index Patterns`<br>

**Create `new_index_pattern` after receiving the first log in `new_index` [🔗Create an index pattern](https://www.elastic.co/guide/en/kibana/7.17/index-patterns.html#settings-create-pattern)**<br>
- **+ Create index pattern**
  - **Name:** `new_alias-*`
  - **Timestamp field:** @timestamp
```diff
+ **Name:** `new_alias`* > use filters to search differents alias, inside index pattern (fe & db...)
+ **Name:** `new_alias` > results docs from just one alias
```

## PIPELINE INGEST JSON LOG<br>

```json
PUT _ingest/pipeline/geoip-pipeline
{
  "description" : "GeoIP message",
  "processors" : [
    {
      "grok" : {
        "field" : "message",
        "patterns" : [
          "%{IP:host.ip} %{WORD:http.request.method} %{URIPATHPARAM:url.original} %{NUMBER:http.request.bytes:int} %{NUMBER:event.duration:double} %{GREEDYDATA}"
        ],
        "ignore_missing" : true
      }
    },
    {
      "geoip" : {
        "field" : "host.ip"
      }
    }
  ]
}
```

## SNAPSHOT

**File System**<br>
```json
PUT _snapshot/fs-snapshots
{
  "type": "fs",
  "settings": {
    "location": "/snap"
  }
}
```

**Snapshot > all indexes**<br>
```json
PUT _slm/policy/full-snapshots
{
  "schedule": "0 /30 * * * ?",
  "name": "<full-snapshot-{now/d}>",
  "repository": "fs-snapshots",
  "config": {
    "indices": "*",
    "include_global_state": true
  },
  "retention": {
    "expire_after": "1d",
    "min_count": 2,
    "max_count": 3
  }
}
```

**Snapshot > Exclude filebeat* Indexes, saves all configs**<br>
```json
PUT _slm/policy/config-snapshots
{
  "schedule": "0 /30 * * * ?",
  "name": "<config-snapshot-{now/d}>",
  "repository": "fs-snapshots",
  "config": {
    "indices": "-filebeat*",
    "include_global_state": true
  },
  "retention": {
    "expire_after": "1d",
    "min_count": 2,
    "max_count": 3
  }
}
```
```json
PUT _slm/policy/snapshot-mod
{
  "schedule": "0 0 12 * * ?",
  "name": "<config-snapshot-mod-{now/d}>",
  "repository": "fs-snapshots",
  "config": {
    "indices": "-filebeat*",
    "include_global_state": true
  },
  "retention": {
    "expire_after": "3d",
    "min_count": 2,
    "max_count": 3
  }
}

PUT _slm/policy/snapshot-eod
{
  "schedule": "0 0 19 * * ?",
  "name": "<config-snapshot-eod-{now/d}>",
  "repository": "fs-snapshots",
  "config": {
    "indices": "-filebeat*",
    "include_global_state": true
  },
  "retention": {
    "expire_after": "3d",
    "min_count": 2,
    "max_count": 3
  }
}
```
```diff
+ **Config Snapshots:** Pipelines, ILM Policies, Index Patterns, Templates, Spaces, Queries and Dashboards 
```

**Execute Snapshot**<br>
```json
POST _slm/policy/full-snapshots/_execute
POST _slm/policy/config-snapshots/_execute
```

**Restore Snapshot**<br>
Restore details:
- [x] All data streams and indices, including system indices
- [x] Restore global state
- [x] Restore aliases

Restored:
- Spaces
- Dashboards
- Index Patterns
- Index Templates
- Components Templates
- Ingest Pipelines
- Saved Objects
- Index Lifecycle Policies

Verify:
- User & Roles

## SEND LOGS

```bash
#!/bin/bash
for i in {1..25000}
do
        NOW=$(date '+%Y-%m-%dT%H:%M:%S.%3NZ')
        curl -H "Content-Type: application/json" -X POST "http://<ip>:9200/server-logs/_doc" -d '{"@timestamp": "'"${NOW}"'","info": "Lorem ipsum dolor sit amet, consectetur adipiscing elit.","environment": "stg"}'
        echo "" 
done

$ chmod +x loop.sh
$ bash loop.sh
```

## EXTRA

**Enable Stack Monitoring**

**Index Pattern Management**
- `.ds-ilm-history-5-*`: Register ILM action's steps
- `.monitoring-es-7-*`: Register Elastic Index and Node Status
- `.monitoring-kibana-7-*`: Register Kibana Status

**Allow/Deny Logs - API ▶︎ `Dev Tools`**<br>
```json
PUT /_cluster/settings
{
  "persistent" : {
    "xpack.security.transport.filter.allow" : "172.16.0.0/24"
  }
}
```
```json
PUT /_cluster/settings
{
  "persistent" : {
    "xpack.security.transport.filter.enabled" : false
  }
}
```
```json
PUT /_cluster/settings
{
  "persistent" : {
    "xpack.security.transport.filter.allow": "192.168.56.1",
    "xpack.security.transport.filter.deny": "192.168.56.0/24"
  }
}
```

**You configure IP filtering by specifying the *xpack.security.transport.filter.allow* and *xpack.security.transport.filter.deny*.**<br>
**Allow rules take precedence over the deny rules**<br>
```yml
xpack.security.transport.filter.allow: "192.168.0.1"
xpack.security.transport.filter.deny: "192.168.0.0/24"
```

**The *_all* keyword can be used to *deny all connections* that are not *explicitly* allowed**<br>
```yml
xpack.security.transport.filter.allow: [ "192.168.0.1", "192.168.0.2", "192.168.0.3", "192.168.0.4" ]
xpack.security.transport.filter.deny: _all
```

**IP filtering configuration also support *IPv6 addresses***<br>
```yml
xpack.security.transport.filter.allow: "2001:0db8:1234::/48"
xpack.security.transport.filter.deny: "1234:0db8:85a3:0000:0000:8a2e:0370:7334"
```

**You can also filter by hostnames when *DNS lookups* are available**<br>
```yml
xpack.security.transport.filter.allow: localhost
xpack.security.transport.filter.deny: '*.google.com'
```

***Disabling IP filtering* can slightly improve performance under some conditions**<br>
**To disable IP filtering entirely:**
```yml
xpack.security.transport.filter.enabled: false
```

**You can also disable IP filtering for the transport protocol but enable it for HTTP only**<br>
```yml
xpack.security.transport.filter.enabled: false
xpack.security.http.filter.enabled: true
```

**Dev Tools > Grok Debugger**<br>
(1) - Sample Data<br>
```yml
[Wed Oct 05 18:37:22.744204 2022] [:error] [pid 12683:tid 139658067420736] [client 192.168.56.124:59696] [client 192.168.56.124] ModSecurity: Access denied with code 403 (phase 2). Matched phrase "nikto" at REQUEST_HEADERS:User-Agent. [file "/etc/modsecurity/crs/rules/REQUEST-913-SCANNER-DETECTION.conf"] [line "56"] [id "913100"] [msg "Found User-Agent associated with security scanner"] [data "Matched Data: nikto found within REQUEST_HEADERS:User-Agent: mozilla/5.00 (nikto/2.1.5) (evasions:none) (test:000562)"] [severity "CRITICAL"] [ver "OWASP_CRS/3.2.0"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-reputation-scanner"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "OWASP_CRS/AUTOMATION/SECURITY_SCANNER"] [tag "WASCTC/WASC-21"] [tag "OWASP_TOP_10/A7"] [tag "PCI/6.5.10"] [hostname "sales.kifarunix.com"] [uri "/index.php"] [unique_id "Yz3O4pMZhpOcYpdhYgoXwQAAAEs"]
```
(1) - Grok Pattern<br>
```yml
\[(?<event_date>%{DAY}\s+%{MONTH}\s+%{MONTHDAY}\s+%{TIME}\s+%{YEAR})\]\s+\[\:(?<log_level>\w+)\]\s+\S+.+client\s+(?<source_ip>%{IP})\]\s+(?<error_message>ModSecurity\S+.+code\s+(?<status_code>%{INT}).+)\s+\[file\s+\"(?<rules_file>\S+.+)\"\]\s+\[line\s+\"(?<rule_line_num>%{INT})\"\]\s+\[id\s+\"(?<rule_id>%{INT})\"\]\s+\[msg\s+\"(?<msg>\S+.+)\"\]\s+\[data\s+\"(?<data>\S+.+)\"\]\s+\[severity\s+\"(?<severity>\w+)\"\]\s+\[ver\s+\"(?<owasp_crs_version>\S+)\"\]\s+(?<tags>\S+.+)\s+\[hostname\s+\"(?<hostname>%{IPORHOST})\"\]\s+\[uri\s+\"(?<uri>\S+.+)\"\]\s+\[unique_id\s+\"(?<unique_id>\S+.+)\"\]

\[(?<event_date>%{DAY}%{SPACE}+%{MONTH}%{SPACE}+%{MONTHDAY}%{SPACE}+%{TIME}%{SPACE}+%{YEAR})\]%{SPACE}+\[\:(?<log_level>%{WORD}+)\]%{SPACE}+%{NOTSPACE}+.+client%{SPACE}+(?<source_ip>%{IP})\]%{SPACE}+(?<error_message>ModSecurity%{NOTSPACE}+.+code%{SPACE}+(?<status_code>%{INT}).+)%{SPACE}+\[file%{SPACE}+\"(?<rules_file>%{NOTSPACE}+.+)\"\]%{SPACE}+\[line%{SPACE}+\"(?<rule_line_num>%{INT})\"\]%{SPACE}+\[id%{SPACE}+\"(?<rule_id>%{INT})\"\]%{SPACE}+\[msg%{SPACE}+\"(?<msg>%{NOTSPACE}+.+)\"\]%{SPACE}+\[data%{SPACE}+\"(?<data>%{NOTSPACE}+.+)\"\]%{SPACE}+\[severity%{SPACE}+\"(?<severity>%{WORD}+)\"\]%{SPACE}+\[ver%{SPACE}+\"(?<owasp_crs_version>%{NOTSPACE}+)\"\]%{SPACE}+(?<tags>%{NOTSPACE}+.+)%{SPACE}+\[hostname%{SPACE}+\"(?<hostname>%{IPORHOST})\"\]%{SPACE}+\[uri%{SPACE}+\"(?<uri>%{NOTSPACE}+.+)\"\]%{SPACE}+\[unique_id%{SPACE}+\"(?<unique_id>%{NOTSPACE}+.+)\"\]

---- Replace all \s+ with %{SPACE}+, \S+ with %{NOTSPACE}+, \d with %{INT}, \w with %{WORD} ----
```

(2) - Sample Data<br>
```yml
[Wed Oct 05 18:37:22.744204 2022] [:error] [client 192.168.56.124:59696] [client 192.168.56.124] [unique_id "Yz3O4pMZhpOcYpdhYgoXwQAAAEs"]
```
(2) - Grok Pattern<br>
```yml
\[(?<event_date>%{DAY}\s+%{MONTH}\s+%{MONTHDAY}\s+%{TIME}\s+%{YEAR})\]\s+\[\:(?<log_level>\w+)\]\s+\S+.+client\s+(?<source_ip>%{IP})\]\s+\[unique_id\s+\"(?<unique_id>\S+.+)\"\]
```

(3) - Sample Data<br>
```yml
0,2018-12-17 22:40:30.000,25980000,92,True,0,400,33,0,2018-12-16 22:40:30.000,2018-12-17 05:53:30.000,433,17,32,1,1,18,2018-12-17,Very Awake
```

(3) - Grok Pattern<br>
```yml
%{NUMBER:ID},%{TIMESTAMP_ISO8601:RecordedDateTimeStamp},%{NUMBER:jobId},%{NUMBER:Efficiency},%{DATA:IsMainSleep},%{NUMBER:MinutesAfterWakeup},%{NUMBER:MinutesAsleep},%{NUMBER:MinutesAwake},%{NUMBER:MinutesToFallAsleep},%{TIMESTAMP_ISO8601:SleepStartTime},%{TIMESTAMP_ISO8601:SleepEndTime},%{NUMBER:TimeInBed},%{NUMBER:RestlessCount},%{NUMBER:RestlessDuration},%{NUMBER:AwakeCount},%{NUMBER:AwakeDuration},%{NUMBER:AwakeningsCount},%{DATA:DateOfSleep},%{DATA:SleepState}$
```

**Change Mapping Existing Index**<br>
```json
########## CURRENT ##########

PUT /filebeat-7.17.3-srv01-000003/_mapping
{
  "properties": {
    "cont.numero": {
      "type": "long"
    }
  }
}

GET filebeat-7.17.3-srv01-000003

########## TEMPLATE ##########

PUT _index_template/filebeat-7.17.3-srv01
{
    "index_patterns" : [
        "filebeat-7.17.3-srv01-*"
    ],
    "template": {
      "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0,
        "index.lifecycle.name": "filebeat-7.17.3-srv01-policy",
        "index.lifecycle.rollover_alias": "filebeat-7.17.3-srv01"
      },
      "mappings": {
        "properties": {
          "created_at": {
            "type": "date",
            "format": "EEE dd MMM yyyy HH:mm:ss Z"
          },
          "cont.status": {
            "type": "long"
          },
          "cont.numero": {
            "type": "long"
        }
      }
    }
  }
}

GET _index_template/filebeat-7.17.3-srv01
```

**Time filter quick ranges**<br>
```json
[
  {
    "from": "now/d",
    "to": "now/d",
    "display": "Today"
  },
  {
    "from": "now-15m",
    "to": "now",
    "display": "Last 15 minutes"
  },
  {
    "from": "now-30m",
    "to": "now",
    "display": "Last 30 minutes"
  },
  {
    "from": "now-1h",
    "to": "now",
    "display": "Last 1 hour"
  },
  {
    "from": "now-24h/h",
    "to": "now",
    "display": "Last 24 hours"
  },
  {
    "from": "now-7d/d",
    "to": "now",
    "display": "Last 7 days"
  },
  {
    "from": "now-15d/d",
    "to": "now",
    "display": "Last 15 days"
  },
  {
    "from": "now-30d/d",
    "to": "now",
    "display": "Last 30 days"
  },
  {
    "from": "now-90d/d",
    "to": "now",
    "display": "Last 90 days"
  },
  {
    "from": "now-180d/d",
    "to": "now",
    "display": "Last 180 days"
  }
]
```

**Elastic Search Cert**
```bash
$ vagrant ssh elk
$ docker exec -it elasticsearch /bin/bash

$ bin/elasticsearch-certutil ca --ca-dn CN=aut.lab
$ bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12 -name "aut-lab"
```

## LINKS
**Indexes**<br>
https://aravind.dev/elastic-data-stream/

**Templates**<br>
https://www.elastic.co/guide/en/elasticsearch/reference/current/dynamic-templates.html

**Management**<br>
https://stackoverflow.com/questions/66236879/get-the-filtered-response-in-elasticsearch-cat-apis<br>
https://linuxhint.com/elasticsearch-shard-list<br>
https://keepgrowing.in/tools/how-to-find-and-diagnose-unassigned-elasticsearch-shards/<br>
https://www.alibabacloud.com/blog/597074<br>
https://www.elastic.co/guide/en/elasticsearch/reference/master/high-cpu-usage.html<br>

**Rollover**<br>
https://www.elastic.co/guide/en/elasticsearch/reference/master/indices-rollover-index.html<br>

**Scalability**<br>
https://www.elastic.co/guide/en/elasticsearch/reference/current/scalability.html<br>
https://dattell.com/data-architecture-blog/elasticsearch-shards-definitions-sizes-optimizations-and-more/<br>
https://www.elastic.co/guide/en/elasticsearch/reference/current/cat-shards.html<br>
https://www.elastic.co/guide/en/elasticsearch/reference/current/size-your-shards.html<br>
https://stackoverflow.com/questions/57957296/elasticsearch-set-default-number-of-replicas-to-0<br>

**Security**<br>
https://www.elastic.co/guide/en/elasticsearch/reference/current/ip-filtering.html<br>
https://www.elastic.co/guide/en/elasticsearch/reference/master/security-basic-setup.html<br>
https://www.ibm.com/docs/en/sle/10.2.0?topic=elasticsearch-enabling-https<br>

**xpack Security**
https://blogs.query.ai/encrypted-communication-on-elastic-server<br>

**Structure**<br>
https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html<br>
https://discuss.elastic.co/t/import-ca-cert-as-privatekeyentry-to-http-keystore-solve-unable-to-create-enrollment-token-error/313780<br>
https://stackoverflow.com/questions/67108012/where-does-elasticsearch-certificates-located<br>
https://github.com/elastic/elasticsearch/issues/32531<br>

**Docker**<br>
https://www.baeldung.com/ops/docker-compose-multiple-commands<br>

**grok**<br>
https://alexmarquardt.com/using-grok-with-elasticsearch-to-add-structure-to-your-data/<br>
https://logz.io/blog/grok-pattern-examples-for-log-parsing/<br>

**Map**<br>
https://www.youtube.com/watch?v=j3CuRYaq22c&ab_channel=Elastic<br>
https://www.elastic.co/guide/en/kibana/current/maps.html<br>
https://discuss.elastic.co/t/kibana-map-no-results-found-documents/271966/3<br>
https://blog.barclayhowe.com/adding-a-custom-geoip-field-to-filebeat-and-elasticsearch-in-elk/<br>
http://man.hubwiz.com/docset/ElasticSearch.docset/Contents/Resources/Documents/www.elastic.co/guide/en/elasticsearch/reference/current/geoip-processor.html<br>

**Snapshot**<br>
https://www.elastic.co/guide/en/elasticsearch/reference/7.17/snapshots-register-repository.html#snapshots-filesystem-repository<br>
https://www.elastic.co/guide/en/elasticsearch/reference/7.17/snapshots-take-snapshot.html<br>
https://www.alibabacloud.com/help/en/elasticsearch/latest/create-manual-snapshots-and-restore-data-from-manual-snapshots<br>
https://stackoverflow.com/questions/54149793/exclude-indices-from-elasticsearch-snapshots<br>
https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshot-restore.html<br>
https://discuss.elastic.co/t/kibana-aliases-conflicting-when-restoring-a-snapshot/295390/4<br>

**Cheat Sheet**<br>
https://logz.io/blog/elasticsearch-cheat-sheet/<br>

**IP List**<br>
https://www.webpagetest.org/addresses.php<br>