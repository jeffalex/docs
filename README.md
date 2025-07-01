# [→ Guía Instalación Docker Turismoi API Global](https://turismoi.github.io/setup/api)

## Para importar tours (dentro del bash del contenedor)

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

#Note: Para crear repositorios de ElasticSearch en inglés, correr los comandos:
I18n.locale = :en
TourDocument::TourRepositoryProxy.repository.create_index! force: true
```

## Crear subdominios en /etc/hosts (máquina local)

```sh
127.0.0.1       resellers.turismoi.test
127.0.0.1       proveedores.turismoi.test
127.0.0.1       saas.turismoi.test
```

## Backup Database

```bash
docker-compose up
docker-compose exec db pg_dump -U postgres app_development >apiglobal.sql
```

## Restore Database

Se puede descargar una copia del apiglobal.sql desde https://drive.google.com/open?id=17Z22k8gzXK0Nfq0ujAzHUtcpwJVbxdYH

```bash
docker-compose up
docker-compose exec -u postgres db psql -c 'DROP DATABASE app_development'
docker-compose exec -u postgres db psql -c 'CREATE DATABASE app_development'
docker container exec -i -u postgres $(docker-compose ps -q db) psql app_development < apiglobal.sql
```

# Populate Countries

- bin/rails countries_importer:import

# Preparar importación de un país (dev)

Editar tours_importer.rake:

```
  'mx' => {
    url: 'http://app:3010/api/v1/api_global_packages.json',
    token: 'COLOCAR_TOKEN_AQUI'
  }
```

con un token de Reseller del país. Si no tienes uno, créalo en el admin_panel desde el país origen.

Ahora podrás importar en español e inglés con

- rails tours_importer:import_from[mx , es]
- rails tours_importer:import_from[mx , en]

# Cómo usar ByeBug:

1. Colocar comando `byebug` en la línea de código que se desea depurar
2. Correr en una terminal: `docker attach api-global_web_1`

# Cómo reimportar tours al buscador

1. Ingresar a una terminal de docker: ```docker-compose run --rm web bash```
2. Correr terminal de Rails: `rails c`
3. Correr el comando para eliminar datos en Español: `I18n.locale = :es; TourDocument::TourRepositoryProxy.repository.create_index! force: true`
4. Correr el comando para eliminar datos en Inglés: `I18n.locale = :en; TourDocument::TourRepositoryProxy.repository.create_index! force: true; exit`
5. Correr tareas rake para importar todos los países e idiomas en dev:

- rails tours_importer:import_from[co,es]
- rails tours_importer:import_from[co,en]
- rails tours_importer:import_from[pe,es]
- rails tours_importer:import_from[pe,en]
- rails tours_importer:import_from[la,es]
- rails tours_importer:import_from[la,en]

# MODO SPEEDY

Con el modo Speedy se podrá correr una versión similar a producción, lo cual reduce el consumo de RAM y CPU y
aumenta la velocidad de carga de los proyectos. El contra es que no se debe usar para desarrollo ya que tiene
las siguientes limitaciones: Los cambios que hagan en código y posiblemente en css no se verán reflejados. Deben reiniciar el server.

Para correr el modo speedy usar este comando:

`docker-compose -f docker-compose-speedy.yml up`