# Observació i Monitorització d'un pipeline a Jenkins

## Preparació prèvia

Aquest projecte és bàsicament un Docker compose per aixecar diferents serveis preconfigurats per a la observació i monitorització de Jenkins.

El propi exemple incorpora un Jenkins i les configuracions del projecte estan preparades per a fer-ne ús, en cas que ja es disposi d'un Jenkins propi només cal canviar les configuracions dels endpoints pertinents. 

En qualsevol cas, és necessari instal·lar i configurar els següents plugins al Jenkins sobre el que es vol configurar la observació i monitorització.

- **Prometheus Metrics**: https://plugins.jenkins.io/prometheus/
   - En aquest cas, s'han preparat els fitxers de configuració amb la configuració per defecte del propi plugin, per tant no és necessari configurar res - tot i que és poden modificar els intervals de publicació de mètriques per exemple o la informació que es desitja que publiqui -.

- **Metrics**: https://github.com/jenkinsci/
  - En aquest cas si que és necessari crear un token des de la configuració de Jenkins, ja que aquest plugin només publica les dades en un endpoint amb la següent forma: `http://jenkins_ip:port/metrics/<keyGenerated>/<metrics>`
 
- **Opentelemetry**:  https://plugins.jenkins.io/opentelemetry/
   - Per aquest plugin si que és necessari configurar l'endpoint del collector, en el aquest cas http://host.docker.internal:4317, i opcionalment; si és desitja un enllaç directe a les traces d'un build des de la pròpia pantalla de Jenkins, és pot configurar una visualització de tipus Jaeger amb el següent endpoint: http://host.docker.internal:8081


Addicionalment serà necessari també configurar un webhook a Slack per a rebre els avisos de les alertes definides.

## Eines

El projecte està composat per un `compose.yml` i els fitxers de configuració de les diferents eines. 

Per arrencar el compose, només cal executar la següent comanda sobre l'arrel del projecte:

```bash
docker compose -f compose.yml up -d
```

Aquest compose aixecarà els següents serveis:

* **Jenkins**: Sobre el que caldrà instal·lar els plugins que ens donaran les dades a observar
* **Prometheus**: Farà scrape de les dades dels plugins de Jenkins. Sobre el prometheus plugin directament, sobre el de metrics desprès d'una transformació.
* **AlertManager**: Rebrà les alertes definides per Prometheus i les publicarà contra un canal de Slack d'un workspace preconfigurat.
* **Grafana**: Conté dos dashboards amb els panells definits per a les mètriques recollides per Prometheus.
* **Json-exporter**: Aplicarà les transformacions necessàries sobre les dades del Metrics pluguins per a que estiguin en un format que Prometheus pugui consumir.
* **Jaeger**: Recollirà les dades a través del plugin de Opentelemetry i publicarà un frontal web per a poder-les analitzar i estudiar.

## Frontals i serveis web

Les diferents eines publicaran serveis i frontals web, a continuació es pot veure els ports i serveis tal i com s'han configurat (és poden modificar a través del `compose.yml`):

Els endpoints d'accés als frontals web aixecats per les diferents eines, des de la màquina on corre el procés Docker:

* **Jaeger**: http://localhost:8081
* **Grafana**: http://localhost:3000
* **Prometheus**: http://localhost:80
* **Jenkins**: http://localhost:8080
* **AlertManager**: http://localhost:9093

Els endpoints d'altres serveis web aixecats per les eines, des de la màquina on corre el procés Docker:

* **Json-exporter** (webservice per a la transformació del json):  http://localhost:7979/probe
* **Jaeger** (webservice per a la recollida de dades otpl) http://localhost:4317


# Modificacions

El projecte incorpora les configuracions necessàries de les diferents eines, les configuracions són força autoexplicatives, a continuació només és fa incís en aquelles que es creuen més importants:

## Prometheus

A Prometheus és necessari configurar els endpoints dels plugins de Jenkins de mètriques. S'han de configurar en l'apartat pertinent del fitxer [prometheus.yml](./prometheus/prometheus.yml):

Als targets dels `static_configs`

```yaml
     # S'ha de configurar amb la IP:port de jenkins on s'ha instal·lat el plugin de prometheus
     - targets: ['host.docker.internal:8080'] >>>>> CANVIA-HO :)
     ...
     - targets: 
      # S'ha de configurar amb la IP:port de jenkins i la key generada al manager per al plugin de metrics!
        - http://host.docker.internal:8080/metrics/Thni4u2v3kBLuGNbsXUcg9um2p8UWvHwmZHJbdX__k7QninbXOoVZIO95KFqZykV/healthcheck >>>>> CANVIA-HO :)
```

## Alertmanager

S'ha de configurar el endpoint del webhook de slack al paràmetre `slack_api` del [alertmanager.yml](./alertmanager/alertmanager.yml).

Sinó és vol notificar via slack, és pot usar el fitxer [alertmanager_exemple_webhook.yml](./alertmanager/alertmanager_exemple_webhook.yml) renombrant-lo o referenciant aquest al [compose.yml](./compose.yaml)

## Grafana

La configuració inicial de Grafana es pot trobar a [custom.ini](./grafana/custom.ini).

Aquí s'indiquen els paths on troba dashboards per defecte, i els datasources, i les i les credencials d'administrador per accedir al frontal web: `user/pwd` -> `albciff/albciff`.

## Jenkins

S'ha afegit un Jenkins al compose, a tall d'exemple per si no es disposa d'un Jenkins propi on fer les proves. En cas d'usar el configurat al `compose.yml`, és necessari fer la configuració prèvia i instal·lar els plugins indicats en la primera part d'aquest readme. 


# Notes addicionals

S'usa `host.docker.internal` per a la interconnexió entre les eines del docker compose ja que tots acaben mappejant amb un port del host i s'interconnecten a través de la IP d'aquest.

A continuació es proposa un pipeline de proves molt bàsic per a començar a jugar i a visualitzar dades a través de les diferents eines:

```groovy
pipeline {
    agent any
    
    stages {
        stage('Stage - Primer') {
            steps {
                // fem un sleep de 2 minuts pq es vegi alguna cosa a grafana
                sh(label: 'testing QA - albciff', script: 'sleep 2m')
            }
        }
    }
}
```