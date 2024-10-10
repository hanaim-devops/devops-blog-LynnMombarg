# Onderzoeksplan <img src="plaatjes/logo.png" width="200" align="right">
In dit document stel ik een plan op voor een DevOps tool die ik ga onderzoeken.

## Onderwerp
Fluentd

## Doelgroep
DevOps engineers en softwareontwikkelaars die op zoek zijn naar een tool voor monitoring in een Docker of Kubernetes omgeving. Of mensen met IT kennis die hun kennis willen verbreden.<br>

<!-- TOC -->

- [Onderzoeksplan](#onderzoeksplan)
    - [Onderwerp](#onderwerp)
    - [Doelgroep](#doelgroep)
    - [Hoofdvraag](#hoofdvraag)
    - [Deelvragen](#deelvragen)

<!-- /TOC --> Ondersteunt veel bronnen en outputs (zoals Elasticsearch, AWS, etc.).
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
