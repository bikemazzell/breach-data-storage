# Structure

- One SQL file / One MySQL table

# Observations

- When imported into MySQL produces arrowbe_user table
- Ends abruptly, looks like partial data
- Primary columns of interest: username, email, password, birthday_search (dob), ipaddress, salt
- Additional columns: aim, icq, yahoo, msn

# Process

- Goal: one JSON file containing the desired fields for each entity/user
  - final file should have the desired format and names for columns (e.g. dob instead of birthday_search, 'YYYY-mm-dd' instead of 'mm-dd-YYYY)
- Optional step: remove unused indices (this will speed up import
	- remove lines 91-95 inclusive, e.g.: `sed -i '91,95d' AndroidForums.com_VB_26-12-2013.sql`
- Load into MySQL
	- `mysql -u username -p -e "create database androidforums;"`
	- `mysql -u username -p androidforums < AndroidForums.com_VB_26-12-2013.sql`
- Remove unwanted columns (e.g. maxposts)
	- `ALTER TABLE androidforums.arrowbe_user DROP COLUMN maxposts;`
- Rename columns (e.g. birthday_search to dob)
	- `ALTER TABLE androidforums.arrowbe_user CHANGE birthday_search dob date DEFAULT '0000-00-00' NOT NULL;` etc.
- Export data to JSON format
	- `echo "select username, email, password, salt, icq, yahoo, msn, skype, dob, ip from arrowbe_user" | mysqlsh --sql --uri username@localhost/androidforums --result-format=json > androidforums.json`
- So far, we have gone from 699MB to 125MB of data that's relevant
- Import into MongoDB
	- `mongoimport --type json --db breaches --collection androidforums --file androidforums.json`
- Refine data in MongoDB
	- Open mongosh shell `mongosh`
	- Switch to appropriate db `use breaches`
	- Create an index as you see fit `db.androidforums.createIndex({"email":1});`
	- Set source `db.androidforums.updateMany({},{$set:{source:"androidforums"}});`
	- Move secondary data into its own sub-document `db.androidforums.updateMany({},[{$set: {other: {icq:"$icq", yahoo:"$yahoo", skype:"$skype", msn:"$msn", dob:"$dob", salt:"$salt"}}}]);`
	- Remove original entries for secondary data `db.androidforums.updateMany({},{$unset: {icq:"", yahoo:"", skype:"", dob:"", salt: "", msn:""}});` 
	- Confirm that data looks good `db.androidforums.find({}).limit(1);`
- Export data out to BSON format
	- `mongodump --db breaches --collection=androidforums --out .`
- Import/merge it into our main collection
	- `mongorestore -d breaches -c sites breaches/androidforums.bson`
- Verify that main collection now contains entities with this source
	- In mongosh `db.sites.find({source:"androidforums"}).limit(1);`
- Back up main collection
	- `mongodump --db breaches --collection=sites --out .`
- Remove or store JSON, BSON, SQL files as need
	
# Takeaways
- Dates and IP addresses are often stored in different formats, normalize these before importing them to unified st
- You could avoid MySQL step altogether by parsing SQL file with various tools, but above approach is simpler (IMO) and only requires that DB to live for a short time
