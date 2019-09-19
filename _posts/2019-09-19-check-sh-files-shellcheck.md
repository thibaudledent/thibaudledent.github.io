---
title: "Verify bash scripts with ShellCheck"
published: true
---

A quick and easy way to verify `.sh` scripts in your current folder with Docker and ShellCheck:

```bash
docker run --rm -v $(pwd):/docker-scripts -it koalaman/shellcheck-alpine /bin/sh -c "find /docker-scripts/ -type f -name '*.sh' | xargs shellcheck"
```

E.g.: 

```bash
In /docker-scripts/sort_json.sh line 4:
mydir=$1
^---^ SC2034: mydir appears unused. Verify use (or export if used externally).
```

And you can search for the issue in their wiki: https://github.com/koalaman/shellcheck/wiki/SC2034.

# References

- [ShellCheck (GitHub)](https://github.com/koalaman/shellcheck)

