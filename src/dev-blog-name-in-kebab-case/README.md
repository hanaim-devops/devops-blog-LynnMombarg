# Fluentd: de universele logger

<img src="plaatjes/logo.png" width="250" align="right" alt="mdbook logo om weg te halen" title="maar vergeet de alt tekst niet">

*[Lynn Mombarg, oktober 2024.](https://github.com/hanaim-devops/LynnMombarg)*
<hr/>

In deze blogpost onderzoek ik hoe Fluentd kan worden toegepast in een applicatie dat draait in Kubernetes. De eigen website (Fluentd, z.d.) legt uit dat Fluentd een open-source data verzamelaar is die de data samenvoegt tot een geheel zodat dit beter te begrijpen is. Fluentd doet dit door alle verzamelde data in een JSON format te zetten. Deze data kan bijvoorbeeld gaan over buffering, outputting of filtering en kan worden verzameld uit meerdere bronnen. De universele data kan daarna naar een output source gestuurd worden zoals Elasticsearch. Die toont de data visueel aan de gebruiker. Deze tool is handig om grip op- en kennis over je systeem te krijgen.
<!-- TOC -->

- [Fluentd: de universele logger](#fluentd-de-universele-logger)
    - [Kubernetes](#kubernetes)
    - [Fluentd gebruiken](#fluentd-gebruiken)
    - [Bronvermelding](#bronvermelding)

<!-- /TOC -->

## Kubernetes

## Fluentd gebruiken
Ik gebruik voor mijn opzet een applicatie die checkt of getallen priemgetallen zijn. Deze applicatie draait in Kubernetes. Voeg een nieuwe YAML file toe: "fluentd-deamonset.yaml". Hierdoor deployed Kubernetes op elke node een Fluentd pod die data kan verzamelen van alle containers in de betreffende node.
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:latest
        env:
        - name: FLUENT_ELASTICSEARCH_HOST
          value: "elasticsearch.logging.svc.cluster.local"  # Elasticsearch service
        - name: FLUENT_ELASTICSEARCH_PORT
          value: "9200"
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

Via de volumeMounts schrijf ik alle logs in een node naar de fluentd container. Daarnaast gebruik ik Elasticsearch om de logs naartoe te sturen.

Fluentd heeft een parser nodig om data te extraheren en te verwerken. De data uit Kubernetes bevat namelijk vaak metadata. Daar wil je op filteren. Ik maak hiervoor een "fluentd.conf" aan.

```
<source>
  @type tail
  path /var/log/containers/*.log
  pos_file /var/log/fluentd-containers.log.pos
  tag kube.*
  format json
</source>

<filter kube.**>
  @type kubernetes_metadata
</filter>

<match **>
  @type elasticsearch
  host "#{ENV['FLUENT_ELASTICSEARCH_HOST']}"
  port "#{ENV['FLUENT_ELASTICSEARCH_PORT']}"
  logstash_format true
  flush_interval 5s
</match>
```

## Bronvermelding

- Fluentd. (z.d.). What is Fluentd? | Fluentd. <https://www.fluentd.org/architecture>