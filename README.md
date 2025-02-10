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
mkdir -p bgc-processing-data/pjm bgc-processing-data/matlab bgc-processing-data/netCDF
mkdir testing-data
```

4. you should now have the following directory structure:

```
- argo-db-backend/
- bgc-processing/
- bgc-processing-data/
   -- pjm/
   -- matlab/
   -- netCDF
- testing-data/
- docker-compose.yml
```

5. create `.env.local` files for `argo-db-backend/` and `bgc-processing/` directories. Copy the `.env.example` file and rename to `.env.local` for each directory. Add values to the empty environmental variable for DB_PASSWORD/POSTGRES_PASSWORD
6. build local docker images: `docker compose build`
7. download example database file: [Download SQL file](https://whoi-my.sharepoint.com/:u:/g/personal/eandrews_whoi_edu/ETMqcrz1txtGuIA70uYhS_cBc1hlbGCPbyJI0TdlYb7SzA?e=o16Q80)
8. seed local database with downloaded data (the `restore` command way take a few minutes):

```
docker compose up -d postgres
docker cp /local/path/to/file/bgc-db.sql.gz postgres:backups/
docker compose exec postgres restore bgc-db.sql.gz
docker compose down
```

## How to update with Git

To update your local code with the latest changes in the the Github repositories, you need to update both the parent repository and the submodules.
Run the following commands from the parent repository directory to update all the repos:

```
git pull
git submodule update --rebase --remote
```

## How to use the Django application

1. `cd` into the `argodb-docker-local` repo directory
2. run `docker compose up` to start the full application stack. (use `docker compose up -d` if you want the containers to run in the background and not display output)

This will start the Django application, the Postgres DB, and the BGC Processing application.

You can access the Django application at: `http://localhost:8000/metadata-admin/`. You can use your current login info from the production site.

You can run Django management commands by executing `docker compose run --rm argodb python manage.py <command_name>`. All commands should be executed in the `argodb-docker-local` directory.

For example, to add a new admin user to access the Django application, you can run:
`docker compose run --rm argodb python manage.py createsuperuser`

## How to use the BGC Processing application

1. When the `bcg-processing` container starts, it will automatically execute the `navis_batch_process.py` function, parsing any test data that you place in the `testing-data` directory. The container follows the same naming conventions as the production application, so data should be in a deployment specific subdirectory using the `wn1234` format as a name. Ex:

```
- testing-data/
  - wn1234/
    - 1234.000.isus
    - 1234.001.log
    - 1234.001.msg

```

2. The container will continue to run after the data parsing is complete. To run the script again, execute the following command:
   `docker compose exec bgc-processing conda run -n bgc_py python navis_batch_process.py`
3. Data will be added to the local Django application as it's parsed. You can can verify the new data at `http://localhost:8000/metadata-admin/`
4. To further debug the application, you can open an interactive shell session on the container by running: `docker exec -it bgc-processing bash`

## How to stop

1. run `docker compose down` in the `argodb-docker-local` directory to stop all containers.
