---
title: "Verify bash scripts with ShellCheck"
published: true
---

A quick and easy way to verify `.sh` scripts in your current folder with Docker and ShellCheck. We will use:

* `docker run -it koalaman/shellcheck-alpine /bin/sh`: to run ShellCheck with `/bin/sh` in a Docker container
* `--rm`: to force Docker to remove the container when the main application finishes
* `-v $(pwd):/docker-scripts`: to mount the current directory in the folder `docker-scripts` of the container
* `find /docker-scripts/ -type f -name '*.sh' | xargs shellcheck`: to check all `*.sh` script and check them with `shellcheck`

Here is the command:

```bash
docker run --rm -v $(pwd):/docker-scripts -it koalaman/shellcheck-alpine /bin/sh -c "find /docker-scripts/ -type f -name '*.sh' | xargs shellcheck"
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

