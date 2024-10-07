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

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    kompose.cmd: C:\ProgramData\chocolatey\lib\kubernetes-kompose\tools\kompose.exe convert
    kompose.version: 1.34.0 (cbf2835db)
  labels:
    io.kompose.service: priemtester
  name: priemtester
spec:
  replicas: 1
  selector:
    matchLabels:
      io.kompose.service: priemtester
  template:
    metadata:
      annotations:
        kompose.cmd: C:\ProgramData\chocolatey\lib\kubernetes-kompose\tools\kompose.exe convert
        kompose.version: 1.34.0 (cbf2835db)
      labels:
        io.kompose.service: priemtester
    spec:
      containers:
        - env:
            - name: DB_PASSWORD
              valueFrom:
                configMapKeyRef:
                  name: my-config
                  key: DB_PASSWORD
            - name: DB_URL
              valueFrom:
                configMapKeyRef:
                  name: my-config
                  key: DB_URL
            - name: DB_USERNAME
              valueFrom:
                configMapKeyRef:
                  name: my-config
                  key: DB_USERNAME
            - name: SPRING_PROFILES_ACTIVE
              value: dev
          image: lynnmombarg/priemtester:latest
          name: priemtester
          ports:
            - containerPort: 8080
              protocol: TCP
      imagePullSecrets:
        - name: my-registry-secret
      restartPolicy: Always

```

Via de volumeMounts schrijf ik alle logs in een node naar de fluentd container. Daarnaast gebruik ik Elasticsearch om de logs naartoe te sturen.

Fluentd heeft een parser nodig om data te extraheren en te verwerken. De data uit Kubernetes bevat namelijk vaak metadata. Daar wil je op filteren. Ik maak hiervoor een "fluentd.conf" aan.

```conf
<source>
  @type kubernetes
  @id input_kubernetes
  @label @KUBERNETES
  @log_level info
  @kube_url https://kubernetes.default.svc:443
  @kube_ca_file /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
  @kube_token_file /var/run/secrets/kubernetes.io/serviceaccount/token
  @kube_namespace default
</source>

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

Daarnaast heb ik voor het runnen van mijn fluentd pod een cluster role en een cluster role binding nodig. Hierin geef ik mezelf de rechten om namespaces te bekijken in mijn cluster.

**ClusterRole.yaml**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: fluentd-pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods", "namespaces"]
    verbs: ["get", "list", "watch"]
```

**ClusterRoleBinding.yaml**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: fluentd-pod-reader-binding
subjects:
  - kind: ServiceAccount
    name: default
    namespace: default
roleRef:
  kind: ClusterRole
  name: fluentd-pod-reader
  apiGroup: rbac.authorization.k8s.io
```



## Bronvermelding

- Fluentd. (z.d.). What is Fluentd? | Fluentd. <https://www.fluentd.org/architecture>