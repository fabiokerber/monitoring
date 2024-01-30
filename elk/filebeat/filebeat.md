# Filebeat

## General

**Commands**
```bash
sudo filebeat modules list
docker rm -f filebeat && docker compose -f /vagrant/files/filebeat-docker-compose.yml up -d && docker logs -f filebeat
```

**Logs Policy**
```json
PUT _ilm/policy/filebeat-7.17.3-srv01-policy
{
    "policy": {
      "phases" : {
        "hot" : {
          "min_age" : "0ms",
          "actions" : {
            "rollover" : {
              "max_docs" : 10000
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
            },
            "shrink" : {
              "number_of_shards" : 1
            }
          }
        },
        "cold" : {
          "min_age" : "2d",
          "actions" : { }
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
}
```

**GeoIP Test (+ Pipeline)**
```json
DELETE _index_template/filebeat-7.17.3-srv01

PUT _component_template/default-replica-component-template
{
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0
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
        },
        "geoip": {
          "properties": {
            "location": { "type": "geo_point" }
          }
        }
      }
    }
  }
}

PUT _index_template/filebeat-7.17.3-srv01-index-template
{
  "template": {
    "settings": {
      "index.lifecycle.name": "filebeat-7.17.3-srv01-policy",
      "index.lifecycle.rollover_alias": "filebeat-7.17.3-srv01"
    }
  },
  "index_patterns": [
    "filebeat-7.17.3-srv01-*"
  ],
  "composed_of": [
    "default-mapping-component-template",
    "default-replica-component-template"
  ]
}

PUT filebeat-7.17.3-srv01-000001
{
  "aliases": {
    "filebeat-7.17.3-srv01": {
      "is_write_index": true
    }
  }
}
```

**+ Create index pattern**
  - **Name:** `new_alias-*`
  - **Timestamp field:** @timestamp

## SIMPLE LOGS

**simple.log**
```bash
#!/bin/bash
for i in {1..25000}
do
        echo 'Sequence:' $i
        echo 'Lorem ipsum dolor sit amet, consectetur adipiscing elit' >> /var/log/simple.log
        echo 'Ut enim ad minim veniam' >> /var/log/simple.log
        echo 'Nemo enim ipsam voluptatem quia voluptas sit aspernatur aut odit aut fugit' >> /var/log/simple.log
done
```

## JSON LOGS & GROK

**geoip-pipeline > ELK Pipeline (MUST CREATE PIPELINE!)**<br>

```bash
#!/bin/bash
for i in {1..25000}
do
        a=(184.169.250.146 35.182.98.197 54.207.159.35 34.240.49.81 3.11.88.57 13.36.154.207 20.203.5.144 18.167.234.221 3.113.237.86 13.54.186.110)
        IP=$(echo ${a[RANDOM%${#a[@]}]})
        echo "$IP GET /index.html 500 0.120 new other stuff" >> /var/log/geoip.log
done
```

**decode-json-fields > filebeat.yml (MUST ACTIVATE!)**
```bash
echo '{"cont.level":"INFO","cont.thread":"epollEventLoopGroup-3-17","cont.logger":"pt.sapo.sapofe.site.ResponseInfo","cont.message":"Request received","cont.environment":"production","cont.project":"pqsapopt","cont.referer":"-","cont.ncache":"hit","cont.contentLenght":"15279","cont.method":"GET","cont.responseTime":"0","cont.host":"sapo.pt","cont.protocolVersion":"HTTP/1.1","cont.userAgent":"kube-probe/1.19","cont.remote":"10.135.8.46","cont.uri":"/","cont.tid":"bce3760c-841e-47b5-a047-2753e606e103","cont.status":"200"}' >> /var/log/decode.log
```

**ResponseTime > Type: long (container log)**<br>
```json
{"log":"{\"timestamp\":\"2023-01-07T13:36:04.102\",\"level\":\"INFO\",\"thread\":\"epollEventLoopGroup-3-27\",\"logger\":\"pt.sapo.sapofe.site.ResponseInfo\",\"message\":\"Request received\",\"environment\":\"production\",\"project\":\"pqsapopt\",\"referer\":\"-\",\"ncache\":\"hit\",\"contentLenght\":\"15273\",\"method\":\"GET\",\"responseTime\":\"{\\\"value\\\":0}\",\"protocolVersion\":\"HTTP/1.1\",\"userAgent\":\"kube-probe/1.19\",\"remote\":\"10.135.8.21\",\"uri\":\"/\",\"tid\":\"1f360588-e5f7-40b6-87e3-e39a5885ab10\",\"status\":\"200\"}\n","stream":"stdout","time":"2023-01-07T13:36:04.103057391Z"}
```

## ~/.kube/config
```bash
apiVersion: v1
kind: Config
clusters:
- name: "conteudos"
  cluster:
    server: "https://rancher2.bk.sapo.pt/k8s/clusters/c-vv8bx"
    certificate-authority-data: "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURlekNDQ\
      W1PZ0F3SUJBZ0lJUm1HVFZUME9VWjh3RFFZSktvWklodmNOQVFFTEJRQXdTekVMTUFrR0ExVUUKQ\
      mhNQ1VGUXhHekFaQmdOVkJBb01FbEJVSUVOdmJYVnVhV05oWTI5bGN5QlRRVEVOTUFzR0ExVUVDd\
      3dFVTBGUQpUekVRTUE0R0ExVUVBd3dIVTBGUVR5QkRRVEFlRncweU1UQXpNRGd4TWpBek1EbGFGd\
      zB6TVRBek1EWXhNakF6Ck1EbGFNRXN4Q3pBSkJnTlZCQVlUQWxCVU1Sc3dHUVlEVlFRS0RCSlFWQ\
      0JEYjIxMWJtbGpZV052WlhNZ1UwRXgKRFRBTEJnTlZCQXNNQkZOQlVFOHhFREFPQmdOVkJBTU1CM\
      U5CVUU4Z1EwRXdnZ0VpTUEwR0NTcUdTSWIzRFFFQgpBUVVBQTRJQkR3QXdnZ0VLQW9JQkFRRGE0b\
      nFBckxPQ3BvL2lWdm96VjZvSnYzQ3NhM21aSXNPdG9Wek9NTXA3CkR0d1pJYlpWcG5YTHkxK2VHM\
      lBsdXQ2YTBEWHl3cnd0eDdJT1JTTW1YQUdLRTdiVmJBV1hwamZjRkkzR0pmK1oKdTBQWTZWdkdzV\
      nMyRXpHUTR6b2wwcmx2aWJ6T0dGTkNyczRYdzNCL1c5dkorUGw1Mm1HbjZpYXBtbURMVmt5QwpFd\
      zRlYnRUVHM5ZXBZRzdTbDRkVnQvdnhXejNGTGxYNVV6b0dVc1E5cEJQUVo4ajlDN2NudVhKVnpYR\
      TBOUlhUCnRhOVN4WHJZV0kyMnZlaWxTZG5PclNQeWljN1ZGakhYbVdxR1g3djhvcTg1MzFPVHNKW\
      m5yWmFjRzVTU3lRTUQKWmRvQzIrb0YzdmUySWY0N0FPR3BzZFNHcVpzN0ZLMkVqRm5RMXhxR04yY\
      2pBZ01CQUFHall6QmhNQjBHQTFVZApEZ1FXQkJTSk5aN0tWZHlTTXBzUlhBMXVzVnJqeklxUFZ6Q\
      VBCZ05WSFJNQkFmOEVCVEFEQVFIL01COEdBMVVkCkl3UVlNQmFBRklrMW5zcFYzSkl5bXhGY0RXN\
      nhXdVBNaW85WE1BNEdBMVVkRHdFQi93UUVBd0lCaGpBTkJna3EKaGtpRzl3MEJBUXNGQUFPQ0FRR\
      UFNOVJQb1hTN041ZnlpZDdvK2RvUlArVkt2TG10bUNyazdNemFIRFhtVE9jZApFMEFsL2RIc0MrT\
      2VOakt5V2lLRjk3NnU2NnJmNlZyeVJCRkFuVXpJM01seGhSd0hLVDZuQ3NyZWd5b0pJeDlxCklKS\
      mc0emoza1NoSFpJV3p0aWRueWhXNUh1VTVRREU4cWZ1c2xkK01PSHF4R3JSdU0rdmdGVVBxNFdiS\
      1ZpMFcKc1RoN2lnR3JNN2EyQWlOMDBhTjFDU2xoNGRKQXkzYnlpb0xkeDNOVmNTRnBzeEdvdElIT\
      2ZkektOeitLamtFaQpqTmdzTVVOKzdTRHhZdDZHdFdsaStWcGtMWXZFNEJ0NC9yN1hPNnd2ckpyV\
      2hTdDlxNGx2MUIrWkFGaHZXUEZxCjJoYzVFNGtWcmJBSldjWHNKZUhDNjZGQUZqajhBQkk5VWtGM\
      0RaeUZhUT09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0="

users:
- name: "conteudos"
  user:
    token: "kubeconfig-u-d77f8.c-vv8bx:xcdstkt4k6z7vshjvnhvj6t69tzhb6dvx6jfgmlc9tr54xwc2mxn29"

contexts:
- name: "conteudos"
  context:
    user: "conteudos"
    cluster: "conteudos"
```

## ConfigMap
```bash
$ kubectl -n logging --context conteudos get cm filebeat-filebeat-daemonset-config -o yaml
$ kubectl -n logging --context conteudos edit cm filebeat-filebeat-daemonset-config -o yaml
$ kubectl -n logging --context conteudos rollout restart daemonset/filebeat-filebeat
$ watch kubectl -n logging --context conteudos get pods -l app=filebeat-filebeat
```

## Links
**Structure**<br>
https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-reference-yml.html<br>
https://www.elastic.co/guide/en/beats/filebeat/current/configuration-template.html<br>

**Install**<br>
https://logit.io/sources/configure/ubuntu/<br>
https://doc.hcs.huawei.com/usermanual/mrs/mrs_01_1501.html<br>
https://logz.io/blog/filebeat-tutorial/<br>

**Docker**<br>
https://github.com/shazChaudhry/docker-elastic/blob/master/filebeat-docker-compose.yml<br>
https://gigi.nullneuron.net/gigilabs/filebeat-elasticsearch-and-kibana-with-docker-compose/<br>

**Structure**<br>
https://discuss.elastic.co/t/filebeat-output-two-conditions/135130/2<br>
https://stackoverflow.com/questions/72188167/filebeat-how-to-create-new-field-from-the-path<br>
https://appliedaiconsulting.com/what-is-filebeat-and-why-is-it-imperative/<br>
https://www.metricfire.com/blog/kubernetes-logging-with-filebeat-and-elasticsearch-part-2/<br>

**Keystore**<br>
https://www.elastic.co/guide/en/beats/filebeat/7.17/keystore.html<br>