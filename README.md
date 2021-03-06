[![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://opensource.org/licenses/MIT)

# AutoR2RML
See the [Data2Services documentation](http://d2s.semanticscience.org/) to run d2s-sparql-operations as part of workflows to generate RDF knowledge graph from structured data.

AutoR2RML automatically generates R2RML mapping files for the following inputs:

* Comma-separated files (.csv)
* Tab-separated files (.tsv)
* Pipe-separated files (.psv)
* SQLite files (.sqlite and .db) 
* Postgres database connection
* MySQL database connection

The RDBMS metadata are retrieved using JDBC to build the mapping file. The text file contents are queries through JDBC using [Apache Drill](https://drill.apache.org). It uses the first row of each file as header, so **make sure the first row of your file is the columns label**. The mapping file should work out of the box and represent generic rdf representations with a unique id representing the filepath and row-number within the file. 

Please note that for Apache Drill empty string values are treated as NULL and every cell value are trimmed.

## Build
```shell
docker build -t autor2rml .
```
## Run

AutoR2RML will generate files to help you map your relational databases, CSV, TSV to a RDF target data model:

* a R2RML `mapping.trig` file (at the location provided using `-o`) to generate generic RDF based on the data structure
* a `template SPARQL query` file for each file/table mapped, to help the user start to write the SPARQL mappings required to transform the generic RDF generated to the target data model. e.g. `pharmgkb_drugs.tsv.rq`

### Docker

#### Using Apache Drill for TSV files

* The file paths provided to AutoR2RML will be used by the Apache Drill container, so make sure you are properly using the same shared volumes (`/data` by default)
* AutoR2RML uses by default the first row of the file as column labels to name the generic RDF properties. Uses `--column-header` to provide column labels if the first row is data.

```shell
# Mappings to System.out
docker run -it --rm --link drill:drill autor2rml -j "jdbc:drill:drillbit=drill:31010" -d /data/pharmgkb_drugs -r

# Mappings to a file
docker run -it --rm --link drill:drill -v /data:/data autor2rml -j "jdbc:drill:drillbit=drill:31010" -o /data/pharmgkb_drugs/mapping.trig -r -d /data/pharmgkb_drugs -b https://w3id.org/d2s/ -g https://w3id.org/d2s/graph/autor2rml

# Provide column header (labels)
docker run -it --rm --link drill:drill -v /data:/data autor2rml -j "jdbc:drill:drillbit=drill:31010" -o /data/pharmgkb_drugs/mapping.trig -r -d /data/pharmgkb_drugs --column-header id,name,genericNames,col4
```

#### Using RDBMS

```shell
## Postgres (run docker)
# Run and load Postgres DB
docker run --name postgres -p 5432:5432 -e POSTGRES_PASSWORD=pwd -d -v /data/autor2rml/:/data postgres
docker exec -it postgres bash
su postgres
psql drugcentral < /data/drugcentral.dump.08262018.sql

# Run autor2rml on DB
docker run -it --rm --link postgres:postgres -v /data:/data autor2rml -j "jdbc:postgresql://postgres:5432/drugcentral" -u postgres -p pwd -o /data/autor2rml/mapping.trig

## SQLite
docker run -it --rm -v /data:/data autor2rml -j "jdbc:sqlite:/data/sqlite/chinook.db" -o /data/sqlite/mapping.trig

```

### JDBC URL examples

```shell
# For Apache Drill
jdbc:drill:drillbit=localhost:31010

# For Postgres
jdbc:postgresql://localhost:5432/database

# For SQLite
jdbc:sqlite:/data/sqlite/GEOmetadb.sqlite
```

### Options

```shell
docker run --rm -it autor2rml --help
```
### IDE run config

To test AutoR2RML in your favorite IDE.

```shell
# Main class
nl.unimaas.ids.autor2rml.autor2rml

# Program arguments for Drill
-j "jdbc:drill:drillbit=localhost:31010" -o /data/pharmgkb_drugs/mapping.trig -d /data/pharmgkb_drugs -r
```