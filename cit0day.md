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
- Each folder/subfolder/file has a predictable naming pattern, with website name followed by a space and a variety of suffixes at the end
- Each individual folder contains different files depending on whether passwords have been dehashed or not

# Process

- Flatten the folder structure - all subfolders can be under one level rather than many (e.g. just subfolders within Cit0day folder)
- Remove all non txt files (unknown and unnecessary) unless they are archvies - we unzip/unrar those
- Remove suffixes everywhere - files and folders - this is relatively easy using a commands like:
  - `find Cit0day/Cit0day\ \[_special_for_xss.is\] -mindepth 1 -type d -exec mv -t Cit0day {}`
  - `find Cit0day -depth -type d -execdir bash -c 'mv "$1" "${1%% *}"' _ {} \;`

# Takeaways
