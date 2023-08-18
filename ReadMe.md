# GraphQL Server using FastAPI

## Create the `.env` file

```
DATABASE_URL=postgresql+psycopg2://postgres:postgres@db:5432/blog_db
DATABASE_URL_LOCAL=postgresql+psycopg2://postgres:postgres@localhost:5432/blog_db
POSTGRES_PASSWORD=postgres
POSTGRES_USER=postgres
POSTGRES_DB=blog_db
PGADMIN_DEFAULT_EMAIL=someemail@gmail.com
PGADMIN_DEFAULT_PASSWORD=postgres
```

Docker containers has their own IP so you shouldn't use `localhost` in the DATABASE_URL instead you should use the service name provided in the `docker-compose.yml` file.

## Getting Started

Initialise the alembic

```
alembic init alembic
```

This command will create `alembic.ini` and `alembic` folder file in the root of your app directory.

### Update the `alembic/env.py` file

Please use the following configuration in your alembic/env.py file

```python
import os
from dotenv import load_dotenv
from logging.config import fileConfig
from sqlalchemy import engine_from_config
from sqlalchemy import pool
from alembic import context
import models

BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
load_dotenv(os.path.join(BASE_DIR, ".env"))

config = context.config
config.set_main_option("sqlalchemy.url", os.environ["DATABASE_URL"])

if config.config_file_name is not None:
    fileConfig(config.config_file_name)

target_metadata = models.Base.metadata
```

### Update the `alembic.ini` file

Update the first block in the .ini file

```
[alembic]
script_location = alembic
prepend_sys_path = .
version_path_separator = os
sqlalchemy.url =
```

### Start the docker services

```
docker-compose up
```

This will start all three services specified in the docker-compose.yml file. In case you see some db connection errors in the app service logs, wait for the db service to start.

### Generate the DB migration

```
docker-compose run app alembic revision --autogenerate -m "New Migration"
```

This will create a new migration file under the /alembic/versions directory

### Apply the migration

```
docker-compose run app alembic upgrade head
```

This will allow alembic to create the required tables in the postgres db

## Validation

Now you can open the Postgres Admin on `http://localhost:5050` and log in into the admin portal

- Register the new DB server (Provide the details you have specified in the `.env` file for the Postgres DB)
- Under schemas>public>tables You will be able to see the `posts` table

## Running GraphQL queries

You can open the GraphiQL on `http://localhost:8000/graphql`

- Create a new post

```graphql
mutation CreateNewPost {
  createNewPost(title: "My first post", content: "this is my first post") {
    ok
  }
}
```

- Query all the posts

```graphql
query {
  allPosts {
    title
  }
}
```
