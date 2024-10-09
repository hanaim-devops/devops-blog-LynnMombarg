# Fluentd: de universele logger

<img src="plaatjes/logo.png" width="250" align="right" alt="fluentd logo" title="Fluentd">

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
    - [Alternatieve tools Competitive analasys, tabel maken](#alternatieve-tools-competitive-analasys-tabel-maken)
        - [Logstash](#logstash)
        - [Vector](#vector)
    - [Voor- en nadelen van Fluentd](#voor--en-nadelen-van-fluentd)
    - [Hands-on](#hands-on)
        - [Van docker naar Kubernetes](#van-docker-naar-kubernetes)
        - [Installatie met Helm](#installatie-met-helm)
    - [Conclusie](#conclusie)
    - [Bronvermelding](#bronvermelding)

<!-- /TOC -->
## Kubernetes

Kubernetes is een open-source platform voor het beheren gecontainerizeerde workloads en services. Het biedt een framework om je gedistribueerde systemen veerkrachtig te maken. Zo kan het automatisch voor je schalen of omgaan met falende onderdelen in je systeem. Het gebruik van Kubernetes is daarom binnen DevOps ook bijna niet meer weg te denken. (<https://kubernetes.io/docs/concepts/overview/>)

De blogpost van Goltsman (2021) beschrijft dat Docker containers in Kubernetes standaard outputs en standaard errors loggen. Docker redirect deze loggings naar een Kubernetes driver. Via Kubectl logs kunnen gebruikers deze logs inzien. Het probleem is echter dat als je een pod verwijdert, alle containers en hun loggings ook verwijderd worden. Dit probleem los je op door een onafhankelijke logger in te bouwen.

De Kubernetes architectuur faciliteert een aantal opties om logs te beheren. Een paar daarvan zijn:

- gebruik van sidecar container die in een app's pod draait.
- gebruik van een node-niveau logging agent die op elke node draait.
- logs direct naar een backend pushen.

In deze blogpost kijk ik dus expliciet naar het gebruik van de Fluentd logging agent die op elke node draait. In Kubernetes heb je type DaemonSet voor een Deployment. Hiermee geef je aan dat op elke node een copy van de logging agent wilt. Een node logging agent is wel gelimiteerd tot de standaard output en -streams van de applicatie.

<p align="center">
  <img src="plaatjes/kubernetes-fluentd.webp" width="320" align="center" alt="simpele overview van systeem" title="Overview"><br>
  (Goltsman, 2021)
</p>

Bovenstaande figuur geeft een node weer die twee pods bevat. Deze pods sturen hun logs naar de Fluentd container. Afhankelijk van de configuratie stuurt de logging agent de logs bijvoorbeeld door naar Elasticsearch, zoals in het plaatje staat. Dit kan ook een andere plugin zijn.

## Logger vs. Monitor

In het landschap van CNCF staat Fluentd onder de categorie Observability.

<p align="center">
  <img src="plaatjes/cncf-landscape.png" align="center" alt="CNCF landscape" title="CNCF landscape"><br>
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

## Alternatieve tools (Competitive analasys, tabel maken)

Er bestaan redelijk wat tools die ongeveer hetzelfde doen als Fluentd. Fluentd is echter wel de grootste die ook door grote bedrijven wordt gebruikt. Denk aan Amazon Web Services en Change.org.

Onderstaande alternatieven zijn te ook vinden op de website van CNCF Landscape (z.d.-b). Ik maak een vergelijking tussen Fluentd en de twee alternatieven aan de hand van tabellen.

### Logstash

<img src="plaatjes/logo-logstash.svg" width="60" align="right" alt="logstash logo" title="Logstash">

"*Logstash is a free and open server-side data processing pipeline that ingests data from a multitude of sources, transforms it, and then sends it to your favorite stash*" (LogStash: Collect, Parse, Transform Logs | Elastic, z.d.).

| FluentD    | Logstash | Stad         |
|------------|----------|--------------|
| Jan        | 25       | Amsterdam    |
| Maria      | 30       | Rotterdam    |
| Ahmed      | 22       | Utrecht      |

### Vector

<img src="plaatjes/logo-vector.png" width="80" align="right" alt="vector logo" title="Vector">

"*Vector is a robust open-source log aggregator developed by Datadog. It empowers you to build observability pipelines by seamlessly fetching logs from many sources, transforming the data as needed, and routing it to your preferred destination* (Better Stack Community, 2024)"

| Naam       | Leeftijd | Stad         |
|------------|----------|--------------|
| Jan        | 25       | Amsterdam    |
| Maria      | 30       | Rotterdam    |
| Ahmed      | 22       | Utrecht      |

## Voor- en nadelen van Fluentd

| Voordelen                                                | Nadelen                                                     |
|----------------------------------------------------------|-------------------------------------------------------------|
| Lightweight en resource-efficient.                       | Handmatige configuratie kan lastig zijn.                    |
| Plugin ecosysteem om functionaliteit toe te voegen.      | Minder grote selectie van plugins dan alternatief Logstash. |
| Zowel gestructureerde als ongestructureerde data parsen. |                                                             |
| Data exporteren naar veel verschillende bestemmingen.    |                                                             |

## Hands-on

In dit onderzoek kijk ik naar twee methoden om Fluentd te installeren in een Kubernetes cluster. In de eerste methode zet ik een docker applicatie met Fluentd op en daarna zet ik die over naar een Kubernetes cluster. Hierin maak ik gebruik van loki en Grafana. In de tweede methode gebruik ik Helm. De logs worden in deze methode niet doorgestuurd naar een andere tool.

### Van docker naar Kubernetes

Voor een opzet van Fluentd maak ik gebruik van een Github repository die een Python applicatie bevat die gaat over planten. Daarbij gebruik ik de loki plugin in combinatie met Fluentd zodat ik gebruik kan maken van Grafana. Loki is de main server die verantwoordelijk is voor het opslaan van de logs en het verwerken van queries. Grafana voert de queries uit en laat de logs visueel zien. Dit doe ik met het oog op het beroepsproduct. Daar maken we namelijk gebruik van Grafana ter monitoring.

Kort samengevat betreft de installatie een Fluentd image die ik overzet naar een Kubernetes deployment.

De eerste stap is om de Github repo te clonen.

```bash
git clone -b microservice-fluentd https://github.com/grafana/loki-fundamentals.git
```

De code bevat een fluentd.conf. Zet de volgende code in het bestand.

```conf
<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>

<match>
  @type loki
  url "http://loki:3100"
  extra_labels {"agent": "fluentd"}
  <label>
    service_name $.service
    instance_id $.instance_id
  </label>
  <buffer>
    flush_internal 10s
    flush_at_shutdown true
    chunk_limit_size 1m
  </buffer>
</match>
```

De source bepaalt waar Fluentd zijn logdata vandaan haalt.

- @type forward: Geeft aan dat Fluentd het forward-protocol gebruikt om logs te ontvangen.
- port 24224: Dit is de poort waarop Fluentd luistert. Standaard is dit 24224 voor de forward-plugin.
- bind 0.0.0.0: Dit betekent dat Fluentd verbindingen accepteert vanaf alle netwerkinterfaces.

De match bepaalt waar de gelogde gegevens naartoe moeten worden verzonden.

- @type loki: Dit geeft aan dat Fluentd Loki gebruikt als bestemming voor de logdata.
- url "<http://loki:3100>": Het URL-adres van de Loki-server die Fluentd gebruikt om logs naar toe te sturen.
- extra_labels {"agent": "fluentd"}: Extra labels die toegevoegd worden aan de logs. In dit geval wordt het label agent: fluentd toegevoegd.

Het label block is een stukje naamgeving. Buffer bevat instellingen voor de buffer van logs.

De code bevat nu alleen nog docker-compose bestanden. Run eerst de docker-compose die Fluentd en Grafana opzet. Daarna de docker-compose die een niveau dieper zit in de directory en de app zelf runt.

```bash
docker compose up
docker compose -f .\greenhouse\docker-compose.yml up -d --build
```

Via <http://localhost:5005> kun je de plantenapp bereiken. Doe hier bijvoorbeeld een login.

Ga daarna naar <http://localhost:3000>. Onder Explore vind je Logs. Je ziet nu verschillende services en de logs die ze hebben binnengekregen. Een voorbeeld van de logs die de main_app heeft binnengekregen zie je hieronder. De bovenstaande (laatste) request die je ziet, is een POST request vanaf mijn laptop (ip-adres 127.20.0.1) om in te loggen.

<img src="plaatjes/grafana-main-app.png" alt="logs in grafana" title="Grafana">

Vanaf hier zet ik de applicatie over naar een Kubernetes cluster. Ik gebruik het Kompose command. Deze gebruik ik voor zowel de buitenste docker-compose als de binnenste docker compose. Doe dit dus twee keer. Vergeet niet om naar de directory te gaan waar de docker-compose in staat.

```bash
kompose convert
```

Output van de kompose convert die de eerste docker-compose omzet:

<img src="plaatjes/kompose-convert-resultaat.png" alt="kompose convert resultaat" title="Kompose convert resultaat">

De meest interessante files hiervan zijn natuurlijk de fluentd-deployment en de configmap. De deployment file zet ik om naar een DaemonSet. Eerder in deze blog geef ik aan dat een dergelijke log aggregator op elke node moet draaien.

De DaemonSet gebruikt de image van Fluentd met loki plugin zodat de data naar Grafana doorgestuurd wordt. De configmap is een mapping van de eerder gemaakte fluentd.conf file. Zo kan Kubernetes de file uitlezen.

```yaml
# fluentd-daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  annotations:
    kompose.cmd: kompose-windows-amd64.exe convert
    kompose.version: 1.34.0 (cbf2835db)
  labels:
    io.kompose.service: fluentd
  name: fluentd
spec:
  selector:
    matchLabels:
      io.kompose.service: fluentd
  template:
    metadata:
      annotations:
        kompose.cmd: kompose-windows-amd64.exe convert
        kompose.version: 1.34.0 (cbf2835db)
      labels:
        io.kompose.service: fluentd
    spec:
      containers:
        - args:
            - fluentd
            - -v
            - -p
            - /fluentd/plugins
          env:
            - name: FLUENTD_CONF
              value: fluentd.conf
          image: grafana/fluent-plugin-loki:main
          name: fluentd
          volumeMounts:
            - mountPath: /fluentd/etc/fluentd.conf
              name: fluentd-cm0
              subPath: fluentd.conf
      restartPolicy: Always
      volumes:
        - configMap:
            items:
              - key: fluentd.conf
                path: fluentd.conf
            name: fluentd-cm0
          name: fluentd-cm0

---

#fluentd-cm0-configmap.yaml
apiVersion: v1
data:
  fluentd.conf: |-
    <source>
      @type forward
      port 24224
      bind 0.0.0.0
    </source>

    <match>
      @type loki
      url "http://loki:3100"
      extra_labels {"agent": "fluentd"}
      <label>
        service_name $.service
        instance_id $.instance_id
      </label>
      <buffer>
        flush_internal 10s
        flush_at_shutdown true
        chunk_limit_size 1m
      </buffer>
    </match>
kind: ConfigMap
metadata:
  annotations:
    use-subpath: "true"
  labels:
    io.kompose.service: fluentd
  name: fluentd-cm0

```

Om de app en de Grafana interface te kunnen bereiken moet je nog wel instellen dat beide services van type LoadBalancer zijn. De configuraties zien er zo uit:

```yaml
# main-app
apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: kompose-windows-amd64.exe convert
    kompose.version: 1.34.0 (cbf2835db)
  labels:
    io.kompose.service: main-app
  name: main-app
spec:
  type: LoadBalancer
  ports:
    - name: "5005"
      port: 5005
      targetPort: 5000
  selector:
    io.kompose.service: main-app

---

# grafana-service
apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: kompose-windows-amd64.exe convert
    kompose.version: 1.34.0 (cbf2835db)
  labels:
    io.kompose.service: grafana
  name: grafana
spec:
  type: LoadBalancer
  ports:
    - name: "3000"
      port: 3000
      targetPort: 3000
  selector:
    io.kompose.service: grafana
```

Daarnaast zet ik alle images die ik nodig heb op mijn Docker Hub. Kubernetes herkent mijn lokale images niet.

Dan wil ik alle pods deployen in mijn cluster. De apply moet dus ook twee keer worden gedaan tenzij je ervoor kiest alle .yaml bestanden in dezelfde folder te plaatsen.

```bash
kubectl apply -f .
```

Et voila! Nu heb je een Kubernetes cluster draaiend die een app bevat met Fluentd als log aggregator en Grafana als monitoring tool.

### Installatie met Helm

Een snelle manier om Fluentd in je Kubernetes cluster te installeren is om Helm te gebruiken. Helm is een package manager voor Kuberenetes. Op deze manier heb je binnen een paar minuten Fluentd pods runnen in je cluster. Vereiste is dat je Helm hebt geinstalleerd op je machine.

Voeg een Helm repo toe waar Fluentd charts gehost worden. (Uitleg bitnami)

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

<img src="plaatjes/logging-pods.png" alt="logging pods" title="Logging pods">

Fluentd geeft vervolgens logs van de applicatie. Onderstaande log gaat over de leaderelection in mijn cluster.

```JSON
2024-10-08 15:24:03 2024-10-08 13:16:31.898678959 +0000 kubernetes.var.log.containers.kube-scheduler-docker-desktop_kube-system_kube-scheduler-f8638c76e167b67996d28f3daf166d369b5685306ee6e1335dee20e1b5c68dd5.log: {
  "log":"I1008 13:16:31.8983341 leaderelection.go:250] attempting to acquire leader lease kube-system/kube-scheduler...\n",
  "stream":"stderr",
  "docker":{
    "container_id":"f8638c76e167b67996d28f3daf166d369b5685306ee6e1335dee20e1b5c68dd5"},
    "kubernetes":{
      "container_name":"kube-scheduler",
      "namespace_name":"kube-system",
      "pod_name":"kube-scheduler-docker-desktop",
      "container_image":"registry.k8s.io/kube-scheduler:v1.30.2",
      "container_image_id":"docker://
      sha256:7820c83aa139453522e9028341d0d4f23ca2721ec80c7a47425446d11157b940",
      "pod_id":"f710756d-0161-4506-a49f-b71285dca0a6",
      "pod_ip":"192.168.65.3",
      "host":"docker-desktop",
      "labels":{
        "component":"kube-scheduler",
        "tier":"control-plane"
      },
        "master_url":"https://10.96.0.1:443/api",
        "namespace_id":"af41635f-986c-43ce-80a0-168ba2c0ff8c",
        "namespace_labels":{
          "kubernetes.io/metadata.name":"kube-system"
        }
    }
  }
```

Het grootste verschil tussen de twee methoden is de snelheid en het gemak waarmee je Fluentd installeert in je cluster. Met Helm heb je dit binnen een mum van tijd opgezet. Via docker en dan naar Kubernetes kost veel tijd. Daarnaast deploy je met Helm rechtstreeks in je cluster zonder een configuratiebestand aan te raken. Bij gebruik van een image moet je eerst zelf een .yaml bestand schrijven die een DaemonSet maakt voor Fluentd. Daarentegen heb je dan wel weer meer controle over je configuratie. Het is dus eigenlijk maar net waar je naar op zoek bent.

## Conclusie

Er zijn meerdere manieren om Fluentd te installeren in je Kubernetes cluster. Je kunt gebruik maken van Helm of zelf een .yaml bestand schrijven. Het kan eenvoudig zijn. Maar het kan ook zeker tijdrovend zijn. De conclusie is eigenlijk dat je voor een methode moet kiezen die het best past bij jouw applicatie.

## Bronvermelding

- CNCF Landscape. (z.d.). Geraadpleegd op 8 okt 2024, van <https://landscape.cncf.io/>
- CNCF Landscape. (z.d.-b). Geraadpleegd op 8 okt 2024, van <https://landscape.cncf.io/guide#observability-and-analysis--observability>
- Datadog. (2021, 3 augustus). Log Aggregation: What it is & How it works | DataDog. Datadog. Geraadpleegd op 8 okt 2024, van <https://www.datadoghq.com/knowledge-center/log-aggregation/>
- Fluentd. (z.d.). What is Fluentd? | Fluentd. Geraadpleegd op 7 okt 2024, van <https://www.fluentd.org/architecture>
- Goltsman, K. (2021, 7 december). Cluster-level Logging in Kubernetes with Fluentd - Supergiant.io - Medium. Medium. Geraadpleegd op 8 okt 2024, van <https://medium.com/kubernetes-tutorials/cluster-level-logging-in-kubernetes-with-fluentd-e59aa2b6093a>
- How to Collect, Process, and Ship Log Data with Vector | Better Stack Community. (2024, 9 januari). Geraadpleegd op 8 okt 2024, van <https://betterstack.com/community/guides/logging/vector-explained/>
- LogStash: Collect, Parse, Transform Logs | Elastic. (z.d.). Elastic. Geraadpleegd op 8 okt 2024, van <https://www.elastic.co/logstash>
- OpenAI. (2024). ChatGPT (8 okt. versie) [Large language model]. Geraadpleegd op 8 okt 2024, van <https://chatgpt.com/c/670501ea-a374-8013-9b92-001743c0c48c>
