# Observacio-Monitoritzacio-Jenkins-Pipeline

Docker compose per aixecar diferents serveis preconfigurats per a la observació i monitorització de Jenkins.

Docker-compose amb la cfg de prometheus, grafana, alert-manager etc per a la observació i monitorització d'un pipeline de Jenkins

# Notes

S'ha de configurar el endpoint del webhook de slack al paràmetre `slack_api` del [alertmanager.yml](./alertmanager/alertmanager.yml).

Sinó és vol notificar via slack, és pot user el fitxer [alertmanager_exemple_webhook.yml](./alertmanager/alertmanager_exemple_webhook.yml) renombrant-lo o referenciant aquest al [compose.yml](./compose.yaml)
