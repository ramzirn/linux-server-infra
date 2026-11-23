# Phase 09 — Monitoring & Observability

## 1. Présentation

Cette phase a pour objectif de mettre en place une infrastructure de monitoring permettant de surveiller l'état du serveur Linux et des conteneurs Docker.

La solution repose sur plusieurs outils complémentaires :

- **Prometheus** : collecte et stockage des métriques.
- **Node Exporter** : exposition des métriques système du serveur Linux.
- **cAdvisor** : collecte des métriques des conteneurs Docker.
- **Grafana** : visualisation des métriques à travers des dashboards.
- **Alertmanager** : réception et gestion des alertes générées par Prometheus.

L'ensemble de cette infrastructure est déployé avec Docker Compose.

## 2. Architecture

### Vue d'ensemble

```text
                    UBUNTU SERVER
                          |
             +------------+------------+
             |                         |
        NODE EXPORTER               cADVISOR
             |                         |
       Métriques Linux          Métriques Docker
             |                         |
             +------------+------------+
                          |
                      PROMETHEUS
                          |
               Collecte et stockage
                          |
                +---------+---------+
                |                   |
             GRAFANA           ALERTMANAGER
                |                   |
          Dashboards          Gestion des alertes
```

### Rôle des composants

| Composant | Rôle | Port |
|---|---|---|
| Prometheus | Collecte et stockage des métriques | 9090 |
| Node Exporter | Métriques système Linux | 9100 |
| cAdvisor | Métriques des conteneurs Docker | 8080 |
| Grafana | Visualisation des métriques | 3000 |
| Alertmanager | Gestion des alertes | 9093 |

Les ports des interfaces Prometheus et Grafana sont publiés uniquement sur `127.0.0.1` du serveur Ubuntu. Les autres composants communiquent sur le réseau Docker interne.

## 3. Arborescence

### Documentation du projet

```text
linux-server-infra/
└── 09.monitoring/
    └── README.md
```

### Configuration opérationnelle sur le serveur

Les fichiers de configuration sont stockés dans `/opt/stacks/monitoring/`.

```text
/opt/stacks/monitoring/
│
├── docker-compose.yml
│
├── prometheus/
│   ├── prometheus.yml
│   └── rules/
│       └── alerts.yml
│
├── alertmanager/
│   └── alertmanager.yml
│
├── grafana/
│   ├── dashboards/
│   └── provisioning/
│       ├── datasources/
│       │   └── prometheus.yml
│       └── dashboards/
│           └── dashboards.yml
│
└── README.md
```

## 4. Déploiement Docker Compose

L'infrastructure est définie dans le fichier :

`/opt/stacks/monitoring/docker-compose.yml`

Les cinq services déployés sont :

- `node-exporter`
- `cadvisor`
- `prometheus`
- `grafana`
- `alertmanager`

### Particularités de configuration

**Node Exporter**

Utilise les montages en lecture seule de `/proc`, `/sys` et du système de fichiers racine pour exposer les métriques du serveur hôte.

**cAdvisor**

Utilise les informations du système et de Docker pour collecter les métriques des conteneurs. Il dispose de permissions élevées (`privileged: true`) nécessaires à sa configuration actuelle.

**Prometheus**

Utilise un volume persistant pour conserver les métriques pendant 15 jours.

**Grafana**

Utilise un volume persistant pour conserver ses données et ses dashboards.

**Alertmanager**

Utilise un volume persistant pour conserver son état.

### Volumes Docker

Trois volumes persistants sont utilisés :

- `prometheus_data`
- `grafana_data`
- `alertmanager_data`

Ils permettent de conserver les données indépendamment du cycle de vie des conteneurs.

### Réseau Docker

Un réseau bridge dédié est utilisé :

`monitoring-network`

Les services communiquent entre eux par leurs noms DNS Docker.

## 5. Configuration Prometheus

Fichier :

`/opt/stacks/monitoring/prometheus/prometheus.yml`

### Paramètres globaux

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
```

- `scrape_interval` : fréquence de collecte des métriques.
- `evaluation_interval` : fréquence d'évaluation des règles d'alertes.

### Targets configurées

| Job | Target |
|---|---|
| prometheus | prometheus:9090 |
| node-exporter | node-exporter:9100 |
| cadvisor | cadvisor:8080 |

Prometheus collecte donc les métriques de lui-même, du serveur Linux et des conteneurs Docker.

### Rétention

La durée de conservation des métriques est configurée à 15 jours.

## 6. Node Exporter — Monitoring système

Node Exporter expose les métriques du système Linux.

Les principales métriques surveillées comprennent :

- Utilisation CPU.
- Utilisation mémoire RAM.
- Espace disque.
- Activité des systèmes de fichiers.
- Activité réseau.
- Charge système.

Les données sont collectées par Prometheus puis exploitées dans Grafana.

### Dashboard installé

Dashboard Grafana Labs :

**Node Exporter Full — ID 1860**

Il permet de visualiser les principales ressources système du serveur.

## 7. cAdvisor — Monitoring Docker

cAdvisor collecte les métriques relatives aux conteneurs Docker.

Les principales métriques comprennent :

- Consommation CPU par conteneur.
- Utilisation mémoire.
- Activité des conteneurs.
- Informations sur les ressources utilisées.

Les métriques sont exposées sur le port interne `8080` et collectées par Prometheus.

### État du dashboard Docker

Un dashboard Grafana Labs (ID 21743) a été testé, mais il n'affichait pas les données attendues.

Les vérifications Prometheus ont néanmoins confirmé la présence des métriques cAdvisor :

```bash
docker exec monitoring-prometheus wget -qO- \
'http://localhost:9090/api/v1/query?query=count(container_cpu_usage_seconds_total)'
```

Résultat obtenu : 47 séries.

```bash
docker exec monitoring-prometheus wget -qO- \
'http://localhost:9090/api/v1/query?query=count(container_memory_working_set_bytes)'
```

Résultat obtenu : 48 séries.

La collecte fonctionne donc. La compatibilité du dashboard reste à corriger.

## 8. Grafana — Visualisation

Grafana est utilisé pour exploiter les métriques collectées par Prometheus.

### Datasource

La datasource Prometheus est configurée automatiquement grâce au provisioning.

Fichier :

`grafana/provisioning/datasources/prometheus.yml`

Configuration :

```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: true
```

L'adresse `http://prometheus:9090` est utilisée parce que Grafana et Prometheus communiquent sur le même réseau Docker.

### Provisioning des dashboards

Le fournisseur de dashboards est défini dans :

`grafana/provisioning/dashboards/dashboards.yml`

Le répertoire prévu pour les dashboards est :

`/var/lib/grafana/dashboards`

### Dashboard installé

- Node Exporter Full — ID 1860.

## 9. Alerting — Prometheus et Alertmanager

Un système de détection des anomalies a été configuré.

### Règles d'alertes

Fichier :

`prometheus/rules/alerts.yml`

Trois règles sont définies.

#### 1. TargetDown

```yaml
- alert: TargetDown
  expr: up == 0
  for: 1m
  labels:
    severity: critical
```

Détecte une cible indisponible pendant au moins une minute.

#### 2. HighCPUUsage

```yaml
- alert: HighCPUUsage
  expr: |
    100 - (avg by (instance) (
      rate(node_cpu_seconds_total{mode="idle"}[5m])
    ) * 100) > 85
  for: 5m
  labels:
    severity: warning
```

Détecte une utilisation CPU supérieure à 85 % pendant cinq minutes.

#### 3. HighMemoryUsage

```yaml
- alert: HighMemoryUsage
  expr: |
    (1 -
      node_memory_MemAvailable_bytes /
      node_memory_MemTotal_bytes
    ) * 100 > 90
  for: 5m
  labels:
    severity: warning
```

Détecte une utilisation mémoire supérieure à 90 % pendant cinq minutes.

### Configuration Alertmanager

Fichier :

`alertmanager/alertmanager.yml`

Alertmanager est configuré pour :

- Recevoir les alertes de Prometheus.
- Regrouper les alertes par nom.
- Attendre 30 secondes avant l'envoi d'un groupe.
- Regrouper les nouvelles alertes toutes les cinq minutes.
- Répéter les notifications toutes les quatre heures.

Un receiver nommé `default-receiver` est défini, mais aucun système de notification externe (email, Telegram, etc.) n'est encore configuré.

**Important :** Alertmanager reçoit et gère les alertes, mais ne transmet pas encore de notifications externes.

## 10. Accès aux interfaces

Pour des raisons de sécurité, les interfaces Prometheus et Grafana sont accessibles uniquement depuis le serveur local.

L'accès depuis Linux Mint se fait à travers un tunnel SSH.

### Accès Grafana

Depuis Linux Mint :

```bash
ssh -L 3000:127.0.0.1:3000 linux-server
```

Puis ouvrir :

`http://localhost:3000`

### Accès Prometheus

Depuis Linux Mint, dans un autre terminal :

```bash
ssh -L 9090:127.0.0.1:9090 linux-server
```

Puis ouvrir :

`http://localhost:9090`

Pour consulter les alertes :

`http://localhost:9090/alerts`

### Identifiants Grafana

Le compte administrateur initial est défini dans le Docker Compose.

- Utilisateur : `admin`
- Mot de passe initial : `admin`

Le mot de passe doit être changé lors de la première connexion.

## 11. Commandes d'administration

Toutes les commandes suivantes sont à exécuter sur le serveur Ubuntu.

### Se placer dans le répertoire

```bash
cd /opt/stacks/monitoring
```

### Démarrer l'infrastructure

```bash
docker compose up -d
```

### Arrêter l'infrastructure

```bash
docker compose stop
```

### Redémarrer l'infrastructure

```bash
docker compose restart
```

### Afficher les conteneurs

```bash
docker compose ps
```

### Consulter les logs

```bash
docker compose logs -f
```

Pour consulter un service spécifique :

```bash
docker compose logs -f prometheus
docker compose logs -f grafana
docker compose logs -f alertmanager
```

### Vérifier les targets Prometheus

```bash
docker exec monitoring-prometheus \
  wget -qO- http://localhost:9090/api/v1/targets
```

### Vérifier les règles d'alertes

```bash
docker exec monitoring-prometheus \
  wget -qO- http://localhost:9090/api/v1/rules
```

### Vérifier Alertmanager

```bash
docker exec monitoring-prometheus \
  wget -qO- http://localhost:9090/api/v1/alertmanagers
```

### Vérifier les alertes reçues

```bash
docker exec monitoring-alertmanager \
  wget -qO- http://localhost:9093/api/v2/alerts
```

## 12. Tests et validation

Les tests suivants ont été effectués pendant cette phase.

| Test | Résultat |
|---|---|
| Démarrage des cinq conteneurs | Réussi |
| Connexion Prometheus aux targets | Réussi |
| Collecte Node Exporter | Réussie |
| Collecte cAdvisor | Réussie |
| Accès Grafana via SSH | Réussi |
| Connexion Grafana à Prometheus | Réussie |
| Affichage du dashboard système | Réussi |
| Chargement des trois règles d'alertes | Réussi |
| Connexion Prometheus à Alertmanager | Réussie |
| Déclenchement de TargetDown | Réussi |
| Compatibilité du dashboard Docker testé | À corriger |

### Test de l'alerte TargetDown

Un test réel a été effectué en arrêtant temporairement Node Exporter :

```bash
docker stop monitoring-node-exporter
```

Après plus d'une minute, l'alerte `TargetDown` est passée à l'état `Firing`.

Cela confirme que Prometheus détecte correctement l'indisponibilité d'une cible.

Pour restaurer Node Exporter :

```bash
docker start monitoring-node-exporter
```

Après la reprise de la collecte et la résolution de l'alerte, son état doit revenir à `Inactive`.

## 13. Sécurité

Plusieurs mesures ont été prises :

- Les interfaces Grafana et Prometheus ne sont pas exposées directement sur le réseau.
- L'accès distant se fait par SSH.
- Les configurations montées dans les conteneurs sont en lecture seule lorsque cela est pertinent.
- Les données persistantes sont stockées dans des volumes Docker dédiés.
- Les communications internes passent par un réseau Docker dédié.

### Point d'attention

cAdvisor utilise actuellement `privileged: true`, ce qui lui accorde des permissions importantes sur l'hôte.

Cette configuration devra être réévaluée dans une future étape de durcissement de la sécurité.

Le mot de passe Grafana initial doit également être remplacé et les secrets ne doivent pas être publiés dans le dépôt Git.

## 14. Améliorations futures

Les améliorations suivantes pourront être réalisées ultérieurement :

- Corriger ou remplacer le dashboard Docker.
- Configurer les notifications email ou Telegram dans Alertmanager.
- Ajouter des dashboards personnalisés.
- Mettre en place des alertes supplémentaires.
- Renforcer les permissions de cAdvisor.
- Ajouter une surveillance de la disponibilité des applications.
- Mettre en place une sauvegarde des données Grafana et Prometheus.

## 15. Résultat final

À la fin de cette phase, une infrastructure de monitoring centralisée a été mise en place avec Docker Compose.

Elle permet de :

- Surveiller les ressources système du serveur Linux.
- Surveiller les ressources consommées par les conteneurs Docker.
- Centraliser les métriques avec Prometheus.
- Visualiser les données avec Grafana.
- Détecter les anomalies avec des règles d'alertes.
- Transmettre les alertes à Alertmanager.
- Accéder aux interfaces de monitoring de manière sécurisée via SSH.

**Phase 09 — Monitoring & Observability : infrastructure déployée et principales fonctionnalités validées.**