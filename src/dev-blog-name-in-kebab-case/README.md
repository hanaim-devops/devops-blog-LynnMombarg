# Fluentd: de universele logger

<img src="plaatjes/logo.png" width="250" align="right" alt="mdbook logo om weg te halen" title="maar vergeet de alt tekst niet">

*[Lynn Mombarg, oktober 2024.](https://github.com/hanaim-devops/LynnMombarg)*
<hr/>
In deze blogpost onderzoek ik hoe Fluentd kan worden toegepast in een applicatie dat draait in Kubernetes. Deze technologie ga ik namelijk toepassen in het beroepsproduct waarmee we de coursefase aflsuiten. De applicatie ontwikkelen we volgens de DevOps ideologie. Ik kijk daarom ook naar het verband tussen DevOps en Fluentd.

De eigen website (Fluentd, z.d.) legt uit dat Fluentd een open-source data verzamelaar is die de data samenvoegt tot een geheel zodat dit beter te begrijpen is. Fluentd doet dit door alle verzamelde data in een JSON format te zetten. Deze data kan bijvoorbeeld gaan over buffering, outputting of filtering en kan worden verzameld uit meerdere bronnen. De universele data kan daarna naar een output source gestuurd worden zoals bijvoorbeeld Elasticsearch of Grafana. Die tonen de data visueel aan de gebruiker. Fluentd is handig om grip op- en kennis over je systeem te krijgen.
<!-- TOC -->

- [Fluentd: de universele logger](#fluentd-de-universele-logger)
    - [Kubernetes](#kubernetes)
    - [Logger vs. Monitor](#logger-vs-monitor)
    - [DevOps principes](#devops-principes)
        - [Automatisering](#automatisering)
        - [Monitoring en Observability](#monitoring-en-observability)
        - [Integratie met CI/CD](#integratie-met-cicd)
    - [Alternatieve tools](#alternatieve-tools)
        - [Logstash](#logstash)
        - [Vector](#vector)
    - [Fluentd gebruiken](#fluentd-gebruiken)
        - [Handmatige installatie](#handmatige-installatie)
        - [Installatie met Helm](#installatie-met-helm)
    - [Pitstop applicatie](#pitstop-applicatie)
    - [Conclusie](#conclusie)
    - [Bronvermelding](#bronvermelding)

<!-- /TOC -->

## Kubernetes

De blogpost van Goltsman (2021) beschrijft dat Docker containers in Kubernetes standaard outputs en standaard errors loggen. Docker redirect deze loggings naar een Kubernetes driver. Via Kubectl logs kunnen gebruikers deze logs inzien. Het probleem is echter dat als je een pod verwijdert, alle containers en hun loggings ook verwijderd worden. Dit probleem los je op door een onafhankelijke logger in te bouwen.

De Kubernetes architectuur faciliteert een aantal opties om logs te beheren. Een paar daarvan zijn:

- gebruik van sidecar container die in een app's pod draait.
- gebruik van een node-niveau logging agent die op elke node draait.
- logs direct naar een backend pushen.

In deze blogpost kijk ik dus expliciet naar het gebruik van de Fluentd logging agent die op elke node draait. In Kubernetes heb je type DaemonSet voor een Deployment. Hiermee geef je aan dat op elke node een copy van de logging agent wilt. Een node logging agent is wel gelimiteerd tot de standaard output en -streams van de applicatie.

<p align="center">
  <img src="plaatjes/kubernetes-fluentd.webp" width="400" align="center" alt="mdbook logo om weg te halen" title="maar vergeet de alt tekst niet"><br>
  (Goltsman, 2021)
</p>

Bovenstaande figuur geeft een node weer die twee pods bevat. Deze pods sturen hun logs naar de Fluentd container. Afhankelijk van de configuratie stuurt de logging agent de logs bijvoorbeeld door naar Elasticsearch, zoals in get plaatje staat. Dit kan ook een andere plugin zijn.

## Logger vs. Monitor

In het landschap van CNCF staat Fluentd onder de categorie Observability. 

<p align="center">
  <img src="plaatjes/cncf-landscape.png" align="center" alt="mdbook logo om weg te halen" title="maar vergeet de alt tekst niet"><br>
  (CNCF Landscape, z.d.)
</p>

"*Observability is the practice and ability of a system to be understood from its external outputs*" (CNCF Landscape, z.d.-b). 

Binnen observability vallen verschillende tools. Monitoring, logging etc. Tools ter monitoring zijn vaak visueel zodat je binnenkomende data makkelijk kunt begrijpen. Loggers worden in het systeem gebouwd om informatie te verkijgen over bijvoorbeeld foutmeldingen of events. Maar waar valt Fluentd dan precies onder? Fluentd zit er eigenlijk tussenin. Het verzamelt de logs en stuurt het door naar een monitoring tool. Je noemt zoiets ook wel een log aggregator. In Datadog (2021) staat dat Log aggregation het proces is van data verzamelen, standaardiseren en samenvoegen om een gestroomlijnde log analyze aan te bieden. Zonder log aggregators moeten developers alle data zelf organiseren, voorbereiden en extraheren.

## DevOps principes

Fluentd valt op allerlei manieren samen met principes van DevOps. Een prompt naar OpenAI (2024) genereert een aantal principes die samengaan met de log aggregator.

### Automatisering

Fluentd automatiseert het proces van logverzameling en -verwerking. Dit vermindert handmatige inspanning en minimaliseert fouten.

### Monitoring en Observability

Een van de kernprincipes van DevOps is het waarborgen van zichtbaarheid in de applicatie en infrastructuur. Fluentd verzamelt logs uit verschillende bronnen (zoals applicaties, servers en cloud-omgevingen) en maakt deze toegankelijk voor monitoringtools. Dit helpt teams bij het snel identificeren van problemen en het verbeteren van de prestaties.

### Integratie met CI/CD

Fluentd kan worden geïntegreerd met Continuous Integration/Continuous Deployment (CI/CD) pipelines om real-time logs en metrics te verzamelen. Dit stelt teams in staat om de prestaties van hun applicaties tijdens de implementatiefase te volgen en eventuele problemen onmiddellijk aan te pakken.

## Alternatieve tools

Er bestaan redelijk wat tools die ongeveer hetzelfde doen als Fluentd. Fluentd is echter wel de grootste die ook door grote bedrijven wordt gebruikt. Denk aan Amazon Web Services en Change.org. 

Onderstaande alternatieven zijn te vinden op de website van CNCF Landscape (z.d.-b).

### Logstash 

<img src="plaatjes/logo-logstash.svg" width="60" align="right" alt="mdbook logo om weg te halen" title="maar vergeet de alt tekst niet">

"*Logstash is a free and open server-side data processing pipeline that ingests data from a multitude of sources, transforms it, and then sends it to your favorite stash*" (LogStash: Collect, Parse, Transform Logs | Elastic, z.d.). 

### Vector

<img src="plaatjes/logo-vector.png" width="80" align="right" alt="mdbook logo om weg te halen" title="maar vergeet de alt tekst niet">

"*Vector is a robust open-source log aggregator developed by Datadog. It empowers you to build observability pipelines by seamlessly fetching logs from many sources, transforming the data as needed, and routing it to your preferred destination* (Better Stack Community, 2024)"

## Fluentd gebruiken

In dit onderzoek kijk ik naar twee methoden om Fluentd te installeren in een cluster. Handmatig en met Helm. Ik gebruik voor mijn opzet een applicatie die checkt of getallen priemgetallen zijn. Deze applicatie draait in Kubernetes.

### Handmatige installatie

Voeg een nieuwe YAML file toe: "fluentd-deamonset.yaml". Hierdoor deployed Kubernetes op elke node een Fluentd pod die data kan verzamelen van alle containers in de betreffende node.

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
    ...
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

Na al deze stappen zou je een Fluentd pod moeten zien in je cluster.

### Installatie met Helm

Een simpele manier om Fluentd in je Kubernetes cluster te installeren is om Helm te gebruiken. Helm is een package manager voor Kuberenetes. Op deze manier heb je binnen een paar minuten Fluentd pods runnen in je cluster. Vereiste is dat je Helm hebt geinstalleerd op je machine.

Voeg een Helm repo toe waar Fluentd charts gehost worden.

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

Optioneel kun je een andere namespace gebruiken voor je pods.

```bash
kubectl create namespace logging
```

Installeer Fluentd met Helm chart.

```bash
helm install fluentd bitnami/fluentd --namespace logging
```

Verifieer de pods.

```bash
kubectl get pods -n logging
```

Je zou dan dit resultaat zien in je terminal:<br>

<img src="plaatjes/logging-pods.png" alt="mdbook logo om weg te halen" title="maar vergeet de alt tekst niet">

## Pitstop applicatie

## Conclusie

## Bronvermelding

- CNCF Landscape. (z.d.). <https://landscape.cncf.io/>
- CNCF Landscape. (z.d.-b). <https://landscape.cncf.io/guide#observability-and-analysis--observability>
- Datadog. (2021, 3 augustus). Log Aggregation: What it is & How it works | DataDog. Datadog. <https://www.datadoghq.com/knowledge-center/log-aggregation/>
- Fluentd. (z.d.). What is Fluentd? | Fluentd. <https://www.fluentd.org/architecture> (Geraadpleegd op 8-10-2024)
- Goltsman, K. (2021, 7 december). Cluster-level Logging in Kubernetes with Fluentd - Supergiant.io - Medium. Medium. <https://medium.com/kubernetes-tutorials/cluster-level-logging-in-kubernetes-with-fluentd-e59aa2b6093a>
- How to Collect, Process, and Ship Log Data with Vector | Better Stack Community. (2024, 9 januari). <https://betterstack.com/community/guides/logging/vector-explained/>
- LogStash: Collect, Parse, Transform Logs | Elastic. (z.d.). Elastic. <https://www.elastic.co/logstash>
- OpenAI. (2024). ChatGPT (8 okt. versie) [Large language model]. <https://chatgpt.com/c/670501ea-a374-8013-9b92-001743c0c48c>
