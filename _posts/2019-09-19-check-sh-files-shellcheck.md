---
title: "Analyze and find bugs in your shell scripts with ShellCheck"
published: true
---

A quick and easy way to check your `.sh` scripts against bugs within the current folder you are working on with Docker and ShellCheck. The command is a combination of:

* `docker run -it koalaman/shellcheck-alpine /bin/sh`: to run ShellCheck with `/bin/sh` in a Docker container
* `--rm`: to force Docker to remove the container when the main application finishes
* `-v $(pwd):/docker-scripts`: to mount the current directory in the folder `docker-scripts` of the container
* `find /docker-scripts/ -type f -name '*.sh' | xargs shellcheck`: to check all `*.sh` script and check them with `shellcheck`

All put together:

```bash
docker run --rm -v $(pwd):/docker-scripts \
  -it koalaman/shellcheck-alpine \
  /bin/sh -c "find /docker-scripts/ -type f -name '*.sh' | xargs shellcheck"
```

Example of output:

```bash
In /docker-scripts/sort_json.sh line 4:
mydir=$1
^---^ SC2034: mydir appears unused. Verify use (or export if used externally).
```

With the reference `SC2034`, you can search for the issue in their wiki ([SC2034](https://github.com/koalaman/shellcheck/wiki/SC2034)).

# References

- [ShellCheck (GitHub)](https://github.com/koalaman/shellcheck)

