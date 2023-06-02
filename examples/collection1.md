# Structure

Collection1 (root level)
+ Collection  #1_BTC combos
	+ 0.txt
	+ 1.txt
	+ 2.txt
	+ ...
+ Collection  #1_EU combos
...

Certain folders contain files with website names in them, e.g.:
+ Collection  #1_NEW combo semi private_Update Dumps
	+ 3.turboavto.com {3.551} [HASH].txt
	+ ...

# Observations

- All files are in CSV format with username,password format (using either colon : or comma , as separator) with some notable exceptions, e.g.:
	- Collection  #1_NEW combo semi private_Private combos/1 (100).txt has binary and other unrelated lines mixed in, e.g.:
	```
	"Este produto inclui software criptogr??fico escrito por Eric Young (eay@cryptsoft.com)." A palavra "criptogr??fico" pode ser exclu??da se as rotinas da biblioteca n??o forem relacionadas a criptografia :-).
   "This product includes cryptographic software written by Eric Young (eay@cryptsoft.com)" The word 'cryptographic' can be left out if the rouines from the library being used are not cryptographic related :-).
   ?????????????? ?????????????? ???????????????? ?????????????????????????????????? ?????????????????????? ??????????????????????, ?????????????????????????? ???????????? ?????????? (Eric Young, eay@cryptsoft.com)??. ?????????? ?????????????????????????????????????? ?????????? ????????????????, ???????? ???????????????????????? ???????????? ???????????????????? ???? ?????????????? ?? ?????????????????????????? :-).
   ????Ce produit comprend un logiciel cryptographique r??dig?? par Eric Young (eay@cryptsoft.com)????. Le terme ????cryptographique???? peut ??tre omis si les programmes de la biblioth??que utilis??s n'ont aucun rapport avec la cryptographie :-).
   ???Dieses Produkt enth??lt eine kryptographische Software, die von Eric Young geschrieben wurde (eay@cryptsoft.com)???. Das Wort ???kryptografisch??? kann weggelassen werden, wenn die aus der Bibliothek genutzten Routinen nicht kryptografiebezogen sind :-).
   ???Este producto contiene software criptogr??fico escrito por Eric Young (eay@cryptsoft.com)??? La palabra ???criptogr??fico??? se puede omitir si las rutinas de la biblioteca que se est?? utilizando no est??n relacionadas con la criptograf??a :-).
   ???Niniejszy produkt zawiera oprogramowanie kryptograficzne autorstwa Erica Younga (eay@cryptsoft.com)???. S??owo ???kryptograficzne??? mo??na pomin????, je??eli wykorzystane fragmenty biblioteki nie dotycz?? kryptografii :-).
   ```
- Other examples include questionable data starting with non-printable/binary characters, non-conforming to email patterns, etc.
- Post processing, file size of CSV is around 33.8GB

# Process

- Goal: all files combined into one, however we wish to keep the metadata, i.e. not just username,password but also the source (website) if it's available
  - final file should look like: `username,password,source` with no duplicates in these combinations
  - this format makes for easy ingestion into our DB
- Remove all empty folders
	- `find Collection1 -type d -empty -delete`
- Remove all non txt files (unknown and unnecessary)
	- `find /path/to/directory -type f ! -name "*.txt" -exec rm -f {} \;`
- Remove crud and normalize all files to have email,pass pattern:
	-  `find Collection1 -type f -exec bash -c 'sed -i -n -E "/^[a-zA-Z0-9._%!$+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}[:|,][^:|^,]*$/p;" "{}"' \;`
- Combine all files within Collection1 folder into one (see **Exception** below)
	- `find Collection1 -type f -exec cat {} \; > collection1-combined.csv`
	
	
- **Exception**: for the few folders that have website names for filenames, move them to a different folder (e.g. `extra`) for additional processing
	- Rename all files to just the website names
		- `find extra -depth -type f -exec bash -c 'mv "$1" "${1%% *}"' _ {} \;`
	- Replace any colons with commas in case of wayward files
		- `find extra -type f -exec bash -c 'sed -i  "s/:/,/g" "{}"' \;`		
	- Append the source (website) to each line following username,password fields
		- `find extra -type f -exec bash -c 'sed -i "s/$/,$(basename "{}")/" "{}"' \;`
- Because this collection will have three vs two fields, we will combine and process it separately from the other; we combine all files into one
	- `find extra -type f -exec cat {} \; > extra-combined.csv`
- If we try to `mongoimport` this csv now, we will get errors like these:
	- `Failed: read error on entry #16591: line 16591, column 30: extraneous " in field`
	- This means a field (most likely password) has a quote it in and MongoDB needs to have these a) doubled and b) the whole field surrounded by quotes
	- We solve this with the following command
		- `sed -i "s/\"/\"\"/g;s/,/\",\"/g;s/^/\"/;s/$/\"/" extra-combined.csv`

# Takeaways
- Examine the files carefully to avoid surprises in form of bad data
	- You don't have to look through every file; combining them into one and doing a sort would allow you to get an idea of outliers on either end
- If a subset has more info, process it separately to retain that extra data

