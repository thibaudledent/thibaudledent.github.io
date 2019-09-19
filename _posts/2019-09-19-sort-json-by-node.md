---
title: "Sort JSON files by attributes"
published: true
---

Sometimes, when you need to compare different JSONs, it is useful to sort them by attribute to have a better diff. It is possible to do this quite simply with the `jq` command. All wrapped in a loop that looks for JSON files in a given folder, and the job is done! (PS: there is a way to simplify this command so that it is only one line long) ;-)

My bash script to sort JSON files by attributes:

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

