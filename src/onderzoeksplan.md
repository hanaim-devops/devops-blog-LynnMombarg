# Onderzoeksplan <img src="plaatjes/logo.png" width="200" align="right">
In dit document stel ik een plan op voor een DevOps tool die ik ga onderzoeken.

## Onderwerp
Fluentd

## Doelgroep
DevOps engineers en softwareontwikkelaars die op zoek zijn naar een tool voor monitoring in een Docker of Kubernetes omgeving. Of mensen met IT kennis die hun kennis willen verbreden.<br>

<img src="plaatjes/software-development-team.jpg" width="200">
*Figuur 1*: <Beschrijving>

## Hoofdvraag
Hoe zet je Fluentd in voor logbeheer in cloud-native omgevingen zoals Kubernetes en Docker?

## Deelvragen
- Wat is Fluentd?
    - Logverzamelingsproces van Fluentd
```
ChatGPT
Fluentd is een open-source data-inzamelings- en logmanagementtool die wordt gebruikt om loggegevens te verzamelen, verwerken en door te sturen naar verschillende opslag- of analysebestemmingen. Het is flexibel, schaalbaar en ondersteunt diverse bronnen en outputs, waardoor het populair is in cloud-native omgevingen zoals Kubernetes. Fluentd helpt organisaties bij het centraliseren van logs uit verschillende systemen voor monitoring, troubleshooting en analyse.
```

- Voor- en nadelen van Fluentd
```
ChatGPT
Voordelen van Fluentd:

    Open-source: Gratis te gebruiken en aanpasbaar.
    Flexibel: Ondersteunt veel bronnen en outputs (zoals Elasticsearch, AWS, etc.).
    Schaalbaar: Geschikt voor zowel kleine als grootschalige infrastructuren.
    Cloud-native: Goed geïntegreerd met Kubernetes en Docker.
    Betrouwbaar: Gebruikt buffering en foutafhandeling voor robuuste logverwerking.

Nadelen van Fluentd:

    Complexe configuratie: Kan lastig zijn om correct op te zetten, vooral bij grotere installaties.
    Prestatieoverhead: Kan meer systeembronnen gebruiken dan lichtere alternatieven.
    Latentie: Mogelijke vertraging bij real-time logverwerking in zeer grote systemen.
```

- Hoe weergeef je overzichtelijk de logs van Fluentd? 
```
ChatGPT
De logging van Fluentd kan overzichtelijk worden weergegeven door het te integreren met visualisatietools zoals Kibana (in combinatie met Elasticsearch) of Grafana (met Loki of Prometheus). Deze tools bieden een gebruiksvriendelijke interface waarmee logs kunnen worden doorzocht, gefilterd en geanalyseerd via dashboards. Daarnaast kunnen ontwikkelaars logstructuren zoals JSON of key-value pairs gebruiken voor duidelijke en gestructureerde logs, waardoor het eenvoudiger wordt om specifieke gebeurtenissen of fouten te vinden.
```

- Alternatieven voor Fluentd
```
ChatGPT
Logstash: Een onderdeel van de Elastic Stack, geschikt voor het verzamelen, verwerken, en verzenden van logs naar Elasticsearch.
Graylog: Een krachtige logmanagementtool met real-time zoek- en analysefuncties.
Loki: Een log aggregator die naadloos integreert met Grafana, geoptimaliseerd voor schaalbaarheid en efficiëntie.
Splunk: Een commerciële oplossing voor logmanagement en analyse, met uitgebreide functies en enterprise support.
Filebeat: Een lichte log shipper van Elastic Stack, voornamelijk voor het verzenden van logs naar Elasticsearch.
```

- Fluentd integreren in testapp