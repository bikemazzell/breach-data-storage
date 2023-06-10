# Structure
- SQL files
- CSV files


# Observations
- After processing data from SQL and importing it to MongoDB, we find some malformed fields
- For example, some email fields are just numbers, without quotes
- Also we find some other fields double quoted, e.g. a field like `email: "'email@here.com'"`
- We need to remove or restructure this bad data to keep our overall collection useful and consistent

# Process

- Goal: all fields corrected and usable
  - final set should have each field in proper format
- In mongosh, remove fields that do not make sense (e.g. numeric passwords that are too short)
	- `db.sites.deleteMany({source:"adultfriendfinder", email:{$type:16}})` or type 1, 18, 19 - see [Mongo types](https://www.mongodb.com/docs/manual/reference/operator/query/type/) for more details
	- above command will remove unquoted numbers in email field
- Export out just the adultfriendfinder collection
	- `mongoexport --db=breaches --collection=sites --query='{ "source": "adultfriendfinder" }' --out=aff.json`
- Use jq tool to remove prefix/suffix single quotes inside a quoted string in username, email, and password fields -- but check if these fields exist first
	- `jq 'if .email? then .email |= gsub("(^'\''|'\''$)"; "") else . end | if .password? then .password |= gsub("(^'\''|'\''$)"; "") else . end | if .username? then .username |= gsub("(^'\''|'\''$)"; "") else . end' aff.json > aff-clean.json`
- Back in mongosh, remove all adultfriendfinder fields
	- `db.sites.deleteMany({source:"adultfriendfinder"})`
- Re-import clean file
	- 
# Takeaways
- Sometimes you miss data issues and have to re-work it
	- This may involve removing it from the overall set first, processing it, and re-importing it back
	- This may require other tools than mongosh (tried replacing w/in mongosh and it took forever!)
- Clean data is important; a search on `test@email.com` will not match a field `'test@email.com'` because of the quotes around the latter
 
