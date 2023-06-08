# ⚒️ breach-data-storage
*Adventures in processing and storing massive amounts of data*

## 🕵️ What this is 
Investigators use any suitable public data for their OSINT investigations. For some of us, this includes breach data, i.e. information released publicly to forums, communities, and channels. A few years ago, it was still possible to obtain a few of these data sets, place them on a fast storage device, and search through them with `ripgrep.` This is no longer the case with daily leaks and major sets reaching upwards of TB of data each.

This repo contains my efforts to organize and optimize this type of data for fast and flexible querying. 

## 🦹‍♂️ What this isn't 
This repo does not and *will not* at any point contain actual data from any breach. It is not intended to encourage any illicit activity. Also, it is not the final word on database design, data transformation, or storage. We are all learning as we go. If you have a better way of doing any of this – reach out and contribute!

## 🧐 Why
With services like HIBP, Dehashed, IntelX, and others, you may wonder why would you ever bother doing this yourself. The answer is it depends. It will take a lot of time to sift through data and organize it. It will likely need money and willingness to spend it on fast data devices. And it will take a mindset to learn and persist through challenges. In return you get a few key benefits: privacy, greater depth of data, and full control of the data.

## ⛰️ Goals
- fast querying of multiple sources of data on key fields, such as `name`, `username`, `email`, `password`, `phone`, `ip`
- querying of additional data that may enrich the above, including source of the data
- low maintenance overhead (e.g. data sources vary in content, all of this needs to co-exist; no server level infrastructure and regular upkeep)
- ease of backup/restore

## 🧰 Tools
[Tools](tools.md) used to process the data

*Note*: My working environment is Debian based Linux. Some commands will need to be adjusted for other variants/flavors.

## 💪 Process
- Analyze the file (type contents, size, etc.)
- If file format is simple (e.g. delimited values), nothing more is necessary at this stage
- If file format is complex (e.g. SQL, JSON), will likely need to import it into a system to manipulate it
  - SQL files most frequently can go into MySQL, unless it was Postgres dump, which would go to Postgres
  - JSON can either be transformed via specialized JSON tools or imported into MongoDB and processed there
- Decide which data will be useful and which can be omitted
  - For SQL, import into a new database, drop unnecessary columns, combine any related data into unified columns, change constraints to allow for NULL values to save space
    - Use `head -100 file.sql` to preview the first 100 lines of the file
    - If a file is far too large to be viewable/editable on your system, consider splitting it up into more manageable chunks, [SQLDumpSplitter](https://philiplb.de/sqldumpsplitter3/) can help in this situation
    - Create a database for the incoming data, e.g. `mysql -u root -p -e "create database db-name;"`
- Import into MongoDB
- Process in MongoDB
  - If fields have not been unified by this stage, do so now
  - If extra fields exist, move them to a sub-document (e.g. `Other` data)
  - Tag data source
  - Remove any un-necessary fields
  - Add indices as necessary
  - Dump out via `mongodump`
  - Restore via `mongorestore` into one unified collection
  - Backup unified collection
  - Test
- TBD: querying from Terminal/Web

## ✍️ Examples
- [Cit0day](examples/cit0day.md) massive collection of older websites and their user credentials
- [Collection1](examples/collection1.md) another large collection of usernames/passwords (mostly w/out website sources and with more noise)
- [COMB](examples/comb.md) yet another very large set of credentials 
- [AndroidForums](examples/androidforums.md) a SQL dump of a popular forum
