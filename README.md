# veld_chain__gutenberg_fuseki

[Project Gutenberg](https://www.gutenberg.org/) doesn't offer a native API and the only existing 
third-party API ( https://gutendex.com/ ) works just on a superficial level. 

However, Project Gutenberg offers its entire metadata as a RDF/XML download: 
https://gutenberg.org/cache/epub/feeds/rdf-files.tar.bz2 .

This repo uses that RDF/XML metadata and ingests it into an Apache Fuseki triplestore, for
arbitrarily complex sparql queries. As such, it can be used for linear workflows where gutenberg
data must be queried in a structured and reproducible way, or it can be adapted into a centralized 
triplestore index for arbitrary remote clients.

All the steps, from download to ingest to querying, are encapsulated as chain velds, described
below.

## requirements

- git
- docker compose

## how to reproduce

clone this repo, with all its submodules (important, as they contain the code modules!):
```
git clone --recurse-submodules https://github.com/veldhub/veld_chain__gutenberg_fuseki.git
```

Then execute the following steps sequentially. See inside their respective VELD yaml files for more 
details.

### download gutenberg metadata

[./veld_download_gutenberg_metadata.yaml](./veld_download_gutenberg_metadata.yaml) : downloads the
aforementioned metadata and extracts it to [./data/gutenberg_rdf/](./data/gutenberg_rdf/).

```
docker compose -f veld_download_gutenberg_metadata.yaml up
```

### run server

[./veld_run_server.yaml](./veld_run_server.yaml) : runs an Apache Fuseki Triplestore server, which
can be reached at http://localhost:3030/ . Its configuration is stored in 
[./data/fuseki_config/](./data/fuseki_config/) and its data at
[./data/fuseki_data/](./data/fuseki_data/) . Important: leave this service running while executing 
the next chains!

```
docker compose -f veld_run_server.yaml up
```

### import rdf

[./veld_import_rdf.yaml](./veld_import_rdf.yaml) : imports the extracted RDF data from the previous
step into the triplestore. Note: this takes a while (on a AMD Ryzen 7 4800H, 32 GB RAM, it takes 
roughly 11 hours) 

```
docker compose -f veld_import_rdf.yaml up
```

### export

[./veld_export.yaml](./veld_export.yaml) : exports data given rq (sparql query) files (samples can
be found in [./data/queries/](./data/queries/)) into supported serializations which are saved into
[./data/fuseki_export/](./data/fuseki_export/)

```
docker compose -f veld_export.yaml up
```

