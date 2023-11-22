# Containerised Jaffle Shop dbt Project

- <u>**Forked from [gmyrianthous/jaffle_shop](https://github.com/gmyrianthous/jaffle_shop)**</u>
- Original Source: [Jaffle Shop dbt project published by dbt Labs](https://github.com/dbt-labs/jaffle_shop)
- If you don't use amd64, recommend to change your image in Dokcerfile.

# Running the project with Docker
You'll first need to build and run the services via Docker (as defined in `docker-compose.yml`):
```bash
$ docker compose up -d --build
```

The commands above will run a Postgres instance and then build the dbt resources of Jaffle Shop as specified in the
repository. These containers will remain up and running so that you can:
- Query the Postgres database and the tables created out of dbt models
- Run further dbt commands via dbt CLI


## Building additional or modified data models
Once the containers are up and running, you can still make any modifications in the existing dbt project 
and re-run any command to serve the purpose of the modifications. 

In order to build your data models, you first need to access the container.

To do so, we infer the container id for `dbt` running container:
```bash
docker ps
```

Then enter the running container:
```bash
docker exec -it <container-id> /bin/bash
```

And finally:

```bash
# Install dbt deps (might not required as long as you have no -or empty- `dbt_packages.yml` file)
dbt deps

# Build seeds
dbt seed --profiles-dir profiles

# Build data models
dbt run --profiles-dir profiles

# Build snapshots
dbt snapshot --profiles-dir profiles

# Run tests
dbt test --profiles-dir profiles
```

Alternatively, you can run everything in just a single command:

```bash
dbt build --profiles-dir profiles
```

You can create a documentation with,
``` bash
dbt docs generate --profiles-dir profiles   # compile your dbt project and warehouse into json files
dbt docs serve --profiles-dir profiles --port 8001  # use these .json files to populate a local website
```


## Querying Jaffle Shop data models on Postgres
In order to query and verify the seeds, models and snapshots created in the dummy dbt project, simply follow the 
steps below. 

Find the container id of the postgres service:
```commandline
docker ps 
```

Then run 
```commandline
docker exec -it <container-id> /bin/bash
```

We will then use `psql`, a terminal-based interface for PostgreSQL that allows us to query the database:
```commandline
psql -h <host> -p <port> -U postgres 
# for my case, psql -h 0.0.0.0 -p 5432 -U postgres
```

You can list tables and views as shown below:
```bash
postgres=# \dt
             List of relations
 Schema |     Name      | Type  |  Owner   
--------+---------------+-------+----------
 public | customers     | table | postgres
 public | orders        | table | postgres
 public | raw_customers | table | postgres
 public | raw_orders    | table | postgres
 public | raw_payments  | table | postgres
(5 rows)

postgres=# \dv
            List of relations
 Schema |     Name      | Type |  Owner   
--------+---------------+------+----------
 public | stg_customers | view | postgres
 public | stg_orders    | view | postgres
 public | stg_payments  | view | postgres
(3 rows)

```

Now you can query the tables constructed form the seeds, models and snapshots defined in the dbt project:
```sql
SELECT * FROM <table_or_view_name>;
```

# DBT Basics

I add dbt basics here for personal studying purposes.

## Project

| Resource                                                     | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [models](https://docs.getdbt.com/docs/build/models)          | Each model lives in a single file and contains logic that either transforms raw data into a dataset that is ready for analytics or, more often, is an intermediate step in such a transformation. |
| [snapshots](https://docs.getdbt.com/docs/build/snapshots)    | A way to capture the state of your mutable tables so you can refer to it later. |
| [seeds](https://docs.getdbt.com/docs/build/seeds)            | CSV files with static data that you can load into your data platform with dbt. |
| [tests](https://docs.getdbt.com/docs/build/tests)            | SQL queries that you can write to test the models and resources in your project. |
| [macros](https://docs.getdbt.com/docs/build/jinja-macros)    | Blocks of code that you can reuse multiple times.            |
| [docs](https://docs.getdbt.com/docs/collaborate/documentation) | Docs for your project that you can build.                    |
| [sources](https://docs.getdbt.com/docs/build/sources)        | A way to name and describe the data loaded into your warehouse by your Extract and Load tools. |
| [exposures](https://docs.getdbt.com/docs/build/exposures)    | A way to define and describe a downstream use of your project. |
| [metrics](https://docs.getdbt.com/docs/build/metrics)        | A way for you to define metrics for your project.            |
| [groups](https://docs.getdbt.com/docs/build/groups)          | Groups enable collaborative node organization in restricted collections. |
| [analysis](https://docs.getdbt.com/docs/build/analyses)      | A way to organize analytical SQL queries in your project such as the general ledger from your QuickBooks. |


## Configuration

Check `dbt_project.yml` file

| YAML key            | Value description                                            |
| ------------------- | ------------------------------------------------------------ |
| name                | Your project’s name in snake case                            |
| version             | Version of your project                                      |
| require-dbt-version | Restrict your project to only work with a range of dbt Core versions |
| profile             | The profile dbt uses to connect to your data platform        |
| model-paths         | Directories to where your model and source files live        |
| seed-paths          | Directories to where your seed files live                    |
| test-paths          | Directories to where your test files live                    |
| analysis-paths      | Directories to where your analyses live                      |
| macro-paths         | Directories to where your macros live                        |
| snapshot-paths      | Directories to where your snapshots live                     |
| docs-paths          | Directories to where your docs blocks live                   |
| vars                | Project variables you want to use for data compilation       |

## Related Links

- https://towardsdatascience.com/jaffle-shop-dbt-docker-93a9b14532a4
- https://docs.getdbt.com/docs/build/projects
- [데이터에 신뢰성과 재사용성까지, Analytics Engineering with dbt](https://tech.socarcorp.kr/data/2022/07/25/analytics-engineering-with-dbt.html)
- [DBT - Best practice guide](https://docs.getdbt.com/best-practices)
- [ETL vs ELT:  5 Critical Differences](https://www.integrate.io/blog/etl-vs-elt/)
- [Gitlab - dbt guide](https://about.gitlab.com/handbook/business-technology/data-team/platform/dbt-guide/#command-line-cheat-sheet)
- [Airflow with DBT 인프라 구성일지](https://velog.io/@jihwankim94/Airflow-with-DBT-인프라-구성일지)
- https://github.com/dbt-labs/dbt-core
- https://github.com/ahnsv/dbt-proof-of-concept
