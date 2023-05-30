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
  - this format makes it easy for ingestion into our DB
- Flatten the folder structure - all sub-folders can be under one level rather than many (e.g. just sub-folders within Cit0day folder)
- Remove all non txt files (unknown and unnecessary) unless they are archives - we unzip/unrar those
- Remove suffixes everywhere - files and folders - this is relatively easy using a commands like:
  - `find Cit0day/Cit0day\ \[_special_for_xss.is\] -mindepth 1 -type d -exec mv -t Cit0day {}`
  - `find Cit0day -depth -type d -execdir bash -c 'mv "$1" "${1%% *}"' _ {} \;`

# Takeaways
