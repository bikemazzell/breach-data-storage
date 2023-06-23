# Structure

Collection2-5&Antipublic (root level)
+ ANTIPUBLIC #1
  + MYR (1).txt
  + MYR (2).txt
  ...
+ ANTIPUBLIC #1_MYRQ ANTIPUBLIC
  + MYR (1).txt
  + MYR (2).txt
  ...
+ Collection #4_Update combos
	+ BTC.txt
	+ BTC_2601300.txt
	...
+ Collection #5_VIP combos
	+ 0.txt
	+ 1.txt
	... 
...
# Observations

- Large collection (500 GB), some user/pass combinations, some email/pass, some phone/pass and some in entirely different format (e.g. just emails)
- Lots of noise, errors, unrelated data
- Many different file formats, structures
- Many duplicates and near-duplicates
- Not one but many different patterns to consider
- Thus far, the longest collection to process
- Resulting set of cleaned data is around 90GB 


# Process

- Unzip and remove zip files
	- `find . -name "*.zip" -execdir 7za x {} \; -exec rm -- {} \;`
- Unrar and remove rar files
	- `find . -name "*.rar" -execdir unrar x {} \; -exec rm -- {} \;`
- Delete 0 byte files
	- `find . -type f -size 0 -delete`
- Delete unrelated files
	- `find . \( -name "*.png" -o -name "*.jpg" -o -name "*.jpeg" -o -name "*.gif" -o -name "*.html" -o -name "*.htm" -o -name "*.js" -o -name "*.swf" -o -name "*.pdf" -o -name "*.py" \) -type f -delete`
- Remove braces suffixes
	- `find . -type f -name '*{*}' -exec sh -c 'for file do mv "$file" "${file%%\{*}"; done' sh {} +`
- Convert XLS/X and XLSX files to CSV -- requires LibreOffice
	- `find . -name "*.xls*" -execdir libreoffice --headless --convert-to "csv:Text - txt - csv (StarCalc):44,34,76,1,1/1" {} \;`
- Convert DOC/X files to TXT - requires LibreOffice
	- `find . -name "*.doc*" -execdir soffice --headless --convert-to txt:"Text (encoded):UTF8" {} \;`
- Clean up XLS/X files
	- `find . -type f -name "*.xls*" -delete`
- Clean up DOC/X files
	- `find . -type f -name "*.doc*" -delete`
- Rename all CSV files to TXT
	- `find . -type f -name "*.csv" -exec sh -c 'mv "$1" "${1%.csv}.txt"' _ {} \;`
- It seems that different files have different end of line marking, we can unify that via dos2unix
	- `find . type f -exec dos2unix {} +`
- Some folders contain files that are named after sites, e.g. xyz.com.txt, we copy those to a different folder for special processing
	- Replace : with , in all files
		- `find . -type f -exec bash -c 'sed -i  "s/:/,/g" "{}"' \`
	- Append the name of the file w/out extension to each line
		- `find . -type f -execdir sh -c 'filename=$(basename "$1"); filename="${filename%.*}"; sed -i "s/$/,$filename/" "$1"' sh {} \;`
- Some folders have a few files all related to the same site, we can unify them as with cit0day, e.g.:
	- `find . -type d -execdir sh -c 'cd "$0" && cat * > combined.txt && sort -u combined.txt -o combined.txt && sed -i "s/:/,/g" combined.txt && sed -i "s/\r/,${PWD##*/}/g" combined.txt' {} \;`
	- `find . -type f -name "combined.txt"  -exec cat {} + > ./unified.csv`
- One folder had some text files without extensions and others with, so we normalize to have TXT extension for all within that folder
	- `find Collection\ #5_DUMP\ dehashed -type f ! -name "*.txt" -exec mv {} {}.txt \;` 
- Another folder had no credentials, just keywords and is safe to remove
	- rm -rf `Коллекция ключевых слов 2015 - разбиты по папкам`
- Folders `Collection #2_New combo cloud_Database Collection_XLSX Base Collection*` and similar contain only sets of names and some phone numbers, mostly Russian; if you do not need it, you can delete them
- Combine all files into one
	- `find . -type f -exec cat {} + > ../c25.csv`
- The rest of the process is clean up specific to your needs, as there is a lot of noisy data in this collection
	- For example, you can focus on just email/password combinations and ignore things that contain only usernames or phone numbers or other unclear data
	- De-duping and removing data that is likely corrupted also takes some time


- SQL files need to be processed in a different way, so we move them out to their own folder
	- `find . -type f -name "*.sql" -exec mv -t /path/to/sql {} +`
- Rest of the process involves bringing in data into MySQL/PostgreSQL, pulling out useful fields into JSON format, and importing them into Mongo
- Example flow
	- `sql mysql -u root -p -e "create database stress;"`
	- `mysql -u root -p stress < stress_forget_tmp.sql`
	- `echo "select login as username, email, hash as password from stress_forget_tmp" | mysqlsh --sql --uri root@localhost/stress --result-format=json > stress.json`
	- `mongoimport --type json --db breaches --collection strss --file stress.json`

