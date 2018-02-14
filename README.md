# Docker + Drupal(Composer) + Elasticsearch

##### Prerequisites:
You need to have [Docker](https://docs.docker.com/install/) and [Docker-compose](https://docs.docker.com/compose/install/) installed.

## Setup
```
cd ~/path_to_projects/$PROJECT/
mkdir src/
docker-compose up -d
sudo chmod -R 777 runtime/elasticsearch
docker-compose down && docker-compose up -d
```
### Elasticsearch
You will need to set "max virtual memory areas" on Your local machine(not Docker).
You may check whether Your elasticsearch container is reporting any errors with:
```
docker logs <CONTAINER_ID>
# OR
docker logs <CONTAINER_NAME>
```
If you are seeing the following message in logs: "virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]",
follow instructions bellow.
```
sudo sysctl -w vm.max_map_count=262144 # temporary solution, will need to be executed upon each initialization of elasticsearch container.
# OR edit sysctl.conf
sudo nano /etc/sysctl.conf
# OR, if You have some other text-editor installed
subl /etc/sysctl.conf
```
Add following lines:
```
### Elasticsearch
# Setting for "max virtual memory areas", system default value is [65530], triggered on elastic bootstrap checks.
vm.max_map_count=262144
```
You may test if everything is working by executing following commands from Your terminal:
```
curl -i http://elasticsearch:9200/_cat/health
curl -i http://elasticsearch:9200/_cluster/health

```

### Drupal
###### Notice: All modules/themes should be required/installed via composer
First we will create our drupal project with composer. This way we will create folder structure
for composer-oriented drupal module management

```
composer create-project drupal-composer/drupal-project:8.x-dev src/ --stability dev --no-interaction -vvv
cd src/
composer require drupal/elasticsearch_connector -o -vvv # With composer require ... you can download new dependencies to your installation.
composer install -o -vvv # The rest of the setup needs to be done in browser or preferably via drupal-console.
```

#### Drupal console
On path: ~/path_to_projects/$PROJECT/src execute the following one-line-greatness:
```php
drupal site:install  standard --langcode="en" --db-type="mysql" --db-host="mariadb" \
       --db-name="drupal" --db-user="drupal" --db-pass="drupal" --db-port="3306" \
       --site-name="Elastic Teenz" --site-mail="danilo@canicinteractive.com" \
       --account-name="admin" --account-mail="danilo@canicinteractive.com" --account-pass="xx" \
       --no-interaction -vvv
```

##### NOTICE:
Do not download/install modules manually or via drush. You can only "enable" them with one of the following:
```
drush en $module_name # drush en elasticsearch_connector
drupal module:install $module_name # drupal module:install elasticsearch_connector
```

#### Drupal >> Elastic-connector setup <Server URL>
<Server URL> for our Elastic-connector is assigned  on drupal local path of /admin/config/search/elasticsearch-connector/edit .
Depending on You local machine setup, the <Server URL> can be assigned to some of the following options: <br />
 *http://localhost:9201/* <br />
 *http://elasticsearch:9200/* <br />
 *http://elastic.$sitename.loc:9200/* ## This is actually the value that we passed in our docker-compose.yml file, see bellow. <br />
 ```yml
 services:
    elasticsearch:
        labels:
            - 'traefik.frontend.rule=Host:elastic.$sitename.loc'
 ```
 *http://CONTAINER_ID:9200/* ## See explanation bellow.
 ```
 docker ps # grab <CONTAINER ID> for our [elasticsearch](docker.elastic.co/elasticsearch/elasticsearch:6.2.1) docker image.
 ```