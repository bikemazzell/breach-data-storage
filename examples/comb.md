# Structure

CompilationOfManyBreaches (root level)
+ data
	+ 1
		+ symbols
		+ 0
		+ 1 
		...
	+ 2
		+ symbols
		+ 0
		+ 1 
		...
	+ a
	...
...


# Observations
- Some malformed lines (e.g. missing password field)
- Quotes in password field require extra processing - doubling them up and surrounding the field with quotes
- No individual sources in this data, so we can add that in mongo post import as `comb` or something similar

# Process

- Goal: all files combined into one, sorted, no duplicates
  - final file should look like: `username,password` with no duplicates in these combinations
- Combine all files into one
	-`find data -type f -exec cat {} \; > comb.csv`
- Sort and remove duplicates
	- `sort -u comb.csv -o comb1.csv`	
- Replace any colons with commas
	- `sed -i  's/:/,/g' comb1.csv`
- Double up existing quoptes and surround fields with quotes
	- `sed -i "s/\"/\"\"/g;s/,/\",\"/g;s/^/\"/;s/$/\"/" comb1.csv`		
- Remove any lines that are missing passwords
	- `sed -i '/,"$/d'`
- Now you can injest the CSV file via `mongoimport`

# Takeaways
- Very large set, takes a while to process, index, backup, etc.
