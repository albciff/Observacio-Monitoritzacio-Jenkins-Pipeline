# Observacio i Monitoritzacio d'un pipeline a Jenkins

Aquest projecte és basicament un docker compose per aixecar diferents serveis preconfigurats per a la observació i monitorització de Jenkins.

Es presuposa la instal·lació dels següents plugins al Jenkins que es preten montitoritzar i observar:

- Prometheus Metrics: https://plugins.jenkins.io/prometheus/

  - Aquest plugin incorpora el Metrics: https://github.com/jenkinsci/metrics-plugin que també es llegira des de prometheus (només healthcheck paràmetres a través del json-exporter)

- OTEL: ...

# Modificacions:

## Prometheus

A Prometheus és necessari configurar els endpoints dels plugins de Jenkins de mertiques. S'han de configurar en l'apartat pertinent del fitxer [prometheus.yml](./prometheus/prometheus.yml):

Als targets dels `static_configs`

```yaml
     # S'ha de configurar amb la IP:port de jenkins on s'ha instal·lat el plugin de prometheus
     - targets: ['10.123.1.30:8080'] >>>>> CANVIA-HO :)
     ...
     - targets: 
      # S'ha de configurar amb la IP:port de jenkins i amb la key generada al manager per al plugin de metrics!
        - http://10.123.1.30:8080/metrics/Thni4u2v3kBLuGNbsXUcg9um2p8UWvHwmZHJbdX__k7QninbXOoVZIO95KFqZykV/healthcheck >>>>> CANVIA-HO :)
```

## Alertmanager

S'ha de configurar el endpoint del webhook de slack al paràmetre `slack_api` del [alertmanager.yml](./alertmanager/alertmanager.yml).

Sinó és vol notificar via slack, és pot user el fitxer [alertmanager_exemple_webhook.yml](./alertmanager/alertmanager_exemple_webhook.yml) renombrant-lo o referenciant aquest al [compose.yml](./compose.yaml)
