# [→ Docker Installation Guide Tourshub API Global](https://tourshub.github.io/setup/api)

## To import tours (inside the container bash)

```sh
rails tours_importer:import_from[co,es]
rails tours_importer:import_from[co,en]
rails tours_importer:import_from[pe,es]
rails tours_importer:import_from[pe,en]
rails tours_importer:import_from[la,en]
rails tours_importer:import_from[SF,es]
rails tours_importer:import_from[SF,en]
bin/rails countries_importer:import
exit
docker-compose up

#Note: To create ElasticSearch repositories in English, run the commands:
I18n.locale = :en
TourDocument::TourRepositoryProxy.repository.create_index! force: true
```

## Create subdomains in /etc/hosts (local machine)

```sh
127.0.0.1       resellers.tourshub.test
127.0.0.1       providers.tourshub.test
127.0.0.1       saas.tourshub.test
```

## Backup Database

```bash
docker-compose up
docker-compose exec db pg_dump -U postgres app_development >apiglobal.sql
```

## Restore Database

You can download a copy of apiglobal.sql from https://drive.google.com/open?id=17Z22k8gzXK0Nfq0ujAzHUtcpwJVbxdYH

```bash
docker-compose up
docker-compose exec -u postgres db psql -c 'DROP DATABASE app_development'
docker-compose exec -u postgres db psql -c 'CREATE DATABASE app_development'
docker container exec -i -u postgres $(docker-compose ps -q db) psql app_development < apiglobal.sql
```

# Populate Countries

- bin/rails countries_importer:import

# Prepare country import (dev)

Editar tours_importer.rake:

```
  'mx' => {
    url: 'http://app:3010/api/v1/api_global_packages.json',
    token: 'PLACE_TOKEN_HERE'
  }
```

with a Reseller token from the country. If you don't have one, create it in the admin_panel from the origin country.

Now you can import in Spanish and English with

- rails tours_importer:import_from[mx , es]
- rails tours_importer:import_from[mx , en]

# How to use ByeBug:

1. Place the `byebug` command on the line of code you want to debug
2. Run in a terminal: `docker attach api-global_web_1`

# How to reimport tours to the search engine

1. Enter a docker terminal: ```docker-compose run --rm web bash```
2. Run Rails terminal: `rails c`
3. Run the command to delete data in Spanish: `I18n.locale = :es; TourDocument::TourRepositoryProxy.repository.create_index! force: true`
4. Run the command to delete data in English: `I18n.locale = :en; TourDocument::TourRepositoryProxy.repository.create_index! force: true; exit`
5. Run rake tasks to import all countries and languages in dev:

- rails tours_importer:import_from[co,es]
- rails tours_importer:import_from[co,en]
- rails tours_importer:import_from[pe,es]
- rails tours_importer:import_from[pe,en]
- rails tours_importer:import_from[la,es]
- rails tours_importer:import_from[la,en]

# MODO SPEEDY

With Speedy mode you can run a version similar to production, which reduces RAM and CPU consumption and
increases project loading speed. The downside is that it should not be used for development as it has
the following limitations: Changes made to code and possibly CSS will not be reflected. You must restart the server.

To run speedy mode use this command:

`docker-compose -f docker-compose-speedy.yml up`