# 🧰 Tools

Depending on the data you are trying to process, you will require different tools. Below are the ones you are likely to encounter, learn, and use. Some come with your system, others you will need to get.

## Data movement tools
  -  `wget` and `curl` for regular http/s sources
  -  [`torsocks`](https://github.com/dgoulet/torsocks) to pull data from onion sites
  -  `rsync` for effective copying or moving of data from one storage to another

## Data processing tools
  - `cut`, `awk`, `sed` along with basic bash script knowledge to string them together
  - `jq` to process JSON files
  - `tr` to clean out noisy text data, e.g. remove all but printable characters in each line
  - [`xsv`](https://github.com/martijn/xsv) to split up large CSV files, get stats, format data
  - [`qsv`](https://github.com/jqnatividad/qsv) similar to above with extra sorting and de-duping options
  
## Database tools

### MySQL
- `mysql` for importing data
- `mysqlsh` for exporting data to JSON format

### Postgres
- `psql` for importing data

### MongoDB
- `mongoimport` for importing CSV/TSV and other similar formats
- `mongosh` for processing data
- `mongodump` for exporting data
- `mongorestore` for restoring or combining data
