---
title: "Sort JSON files by attributes"
published: true
---

A bash script to sort JSON files by attributes:

```bash
#!/bin/bash
set -eEuxo pipefail

mydir=$1
count=0

while IFS= read -r -d '' file
do 
	(( count++ ))
    jq -S '.' "$file" > "$file"_sorted;
    mv "$file"_sorted "$file";
done < <(find "$mydir" -name '*.json' -print0)

echo "Updated $count files."
```

Usage:

```bash
./sort_json.sh /path/to/my/directory
```

# References

- [jq command (GitHub)](https://stedolan.github.io/jq/)

