# Structure

Cit0day (root level)
+ Cit0day Prem [_special_for_xss.is]
  + website {number}(Category)_special_for_xss.is
    + website {number}[class].txt
+ Cit0day [_special_for_xss.is]
  + website {number}(Category)_special_for_xss.is
    + website {number}[class].txt

# Observations

- All files are in CSV format with username:password format
- Each folder/sub-folder/file has a predictable naming pattern, with website name followed by a space and a variety of suffixes at the end
- Each individual folder contains different files depending on whether passwords have been de-hashed or not

# Process

- Goal: all files combined into one, however we wish to keep the metadata, i.e. not just username:password but also the source (website) that comes from the folder name
  - final file should look like: `username,password,source` with no duplicates in these combinations
  - this format makes for easy ingestion into our DB
- Flatten the folder structure - all sub-folders can be under one level rather than many (e.g. just sub-folders within Cit0day folder)
	- `find Cit0day/Cit0day\ \[_special_for_xss.is\] -mindepth 1 -type d -exec mv -t Cit0day {}`
- Remove all empty folders
	- `find Cit0day -type d -empty -delete`
- Remove all non txt files (unknown and unnecessary)
	- `find /path/to/directory -type f ! -name "*.txt" -exec rm -f {} \;`
- Remove suffixes everywhere:
  - `find Cit0day -depth -type d -execdir bash -c 'mv "$1" "${1%% *}"' _ {} \;`
- Combine all files within each subfolder into one file called combined.txt, sort and remove duplicates, and append the name of the folder to the end of each line
	- `find Cit0day -type d -execdir sh -c 'cd "$0" && cat * > combined.txt && sort -u combined.txt -o combined.txt && sed -i "s/:/,/g" combined.txt && sed -i "s/\r/,${PWD##*/}/g" combined.txt' {} \;`
- Combine all of those combined.txt files into one and move it to root folder
	- `find Cit0day -type f -name "combined.txt"  -exec cat {} + > Cit0day/cit0day.csv`
- Remove all of those previously created combined.txt files from individual subfolders
	`find Cit0day -type d -execdir sh -c 'cd "$0" && find . -name "combined.txt" -delete' {} \;`

# Takeaways
- Know what you are after before you begin; your requirements may not match someone else's
- Be prepared for mistakes! Try on smaller subsets of folders/files to test before running on entire set
