---
title: "Sort Maven pom.xml files"
published: true
---

I already had an article to sort `json` files. This time, I'm tackling maven `pom.xml` files!

Indeed, it is sometimes interesting to compare plugins and dependencies of two projects (e.g. to see if all plugins are present in a new project, if all configuration are there...). Comparing the `pom.xml` of the projects is one option, and it's easier to do when the `pom`'s are sorted. 

To sort a `pom.xml`, I found a small maven plugin:

```bash
mvn com.github.ekryd.sortpom:sortpom-maven-plugin:sort \
    -Dsort.keepBlankLines \
    -Dsort.predefinedSortOrder=custom_1 \
    -Dsort.nrOfIndentSpace=4 \
    -Dsort.createBackupFile=false
```

# References

- [Ekryd/sortpom: Maven plugin that helps the user sort pom.xml](https://github.com/Ekryd/sortpom)
- [How to sort dependencies in a section of a Maven POM file - Stack Overflow](https://stackoverflow.com/questions/58080460/how-to-sort-dependencies-in-a-section-of-a-maven-pom-file)
