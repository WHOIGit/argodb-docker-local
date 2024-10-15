# argodb-docker-local

Dockerized local dev environment to test Argo DB django app with BGS data processing app

## Prerequisites

Install Docker on your computer: [https://docs.docker.com/engine/install/](https://docs.docker.com/engine/install/)

## How to install

1. copy this repo to your local computer: `git clone https://github.com/WHOIGit/argodb-docker-local.git`
2. move into this repo directory `cd argodb-docker-local`
3. install sub-module repos:

```
git submodule init
git submodule update
```

5. create two new local directories for testing data and output files:

```
mkdir bgc-processing-data
mkdir testing-data
```

4. you should now have the following directory structure:

```
- argo-db-backend/
- bgc-processing/
- bgc-processing-data/
- testing-data/
- docker-compose.yml
```

5. set up .env.local files for argo-db-backend/ and bgc-processing/. Copy the `.env.example` files
6. build local docker images: `docker compose build`
7. seed local database with real data (the `restore` command way take a few minutes):

```
docker compose up -d postgres
docker cp bgc-db.sql.gz postgres:backups/
docker compose exec postgres restore bgc-db.sql.gz
docker compose down
```

## How to use the Django application

1. `cd` into the `argodb-docker-local` repo directory
2. run `docker compose up` to start the full application stack. (use `docker compose up -d` if you want the containers to run in the background and not display output)

This will start the Django application, the Postgres DB, and the BGC Processing application.

You can access the Django application at: `http://localhost:8000/metadata-admin/`. You can use your current login info from the production site.

You can run Django management commands by executing `docker compose run --rm argodb python manage.py <command_name>`. All commands should be executed in the `argodb-docker-local` directory.

## How to use the BGC Processing application

1. When the `bcg-processing` container starts, it will automatically execute the `navis_batch_process.py` function, parsing any test data that you place in the `testing-data` directory. The container follows the same naming conventions as the production application, so data should be in a deployment specific subdirectory using the `wn1234` format as a name. Ex:

```
- testing-data/
  - wn1234/
    - 1234.000.isus
    - 1234.001.log
    - 1234.001.msg

```

2. The container will stop after the data parsing is complete. To run it again, simply start it up again:
   `docker compose up bgc-processing`
3. Data will be added to the local Django application as it's parsed. You can can verify the new data at `http://localhost:8000/metadata-admin/`

## How to stop

1. run `docker compose down` in the `argodb-docker-local` directory to stop all containers.
