---
title: "Creating a Bitbucket pipeline to automate a Maven release"
published: true
---

It's quite paradoxical to think that my first blog post on GitHub is about Bitbucket... But well, that's the most interesting thing I have to write now and, as a dev, we're curious and ready to try something new, aren't we?

# Context

To put things in context, I recently had to apply changes in a [Java library](https://bitbucket.org/chochos/j8583) (hosted to Bitbucket) that we use on a project at work. Even if the repo maintainer has merged two of my Pull Requests with the main changes, he did not release a new version of his library to Maven Central. The changes were therefore not available for our project.

To speed things up, I decided to [fork the repo](https://bitbucket.org/thibaudledent/j8583). In addition to being able to apply our fixes more quickly, we were also able to take control of the library's release cycle.

# Challenge

Having the control of the repository is one thing but I discovered that making a new Maven release of a library is a process in itself (and you have to store passwords, tokens and GPG key somewhere...). I didn't want to remember this process, nor to repeat it manually several times. So I tried to automate it.

Like many code hosting platform, Bitbucket has its own integrated CI/CD feature called [Bitbucket pipelines](https://bitbucket.org/product/features/pipelines). For example, it is quite useful to create one pipeline to run and test your application.

You can maintain your pipelines in a [YAML file](https://bitbucket.org/thibaudledent/j8583/src/0bdff5d60a2aa9644025da8d0db09f9ba5ff63e2/bitbucket-pipelines.yml?at=master&fileviewer=file-view-default) called `bitbucket-pipelines.yml`. The challenge was to create a custom pipeline to release a new version of the library to Maven Central!

# Steps

## Create a Sonatype User account

1. Go to [Create your JIRA account](https://issues.sonatype.org/secure/Signup%21default.jspa) and create an account for yourself.
2. [Create a New Project ticket](https://issues.sonatype.org/secure/CreateIssue.jspa?issuetype=21&pid=10134) (link to my [Jira ticket](https://issues.sonatype.org/browse/OSSRH-44740))

## Make sure your Group Id and other pom details are correct

 Make sure you use the correct `groupId` in your `pom.xml`, example:

```xml
<groupId>org.bitbucket.thibaudledent.j8583</groupId>
```

Make sure your `pom.xml` (link [here](https://bitbucket.org/thibaudledent/j8583/src/0bdff5d60a2aa9644025da8d0db09f9ba5ff63e2/pom.xml?at=master&fileviewer=file-view-default)) includes a *licenses*, *scm*, *distributionManagement*, and *developers* section. Take note of the *snapshotRepository* and the *repository*. Example:

```xml
<url>https://bitbucket.org/thibaudledent/j8583</url>

<licenses>
    <license>
        <name>GNU Lesser General Public License, version 3</name>
        <url>http://www.gnu.org/licenses/lgpl-3.0.html</url>
        <distribution>repo</distribution>
    </license>
</licenses>

<scm>
    <connection>scm:git: https://bitbucket.org/thibaudledent/j8583.git</connection>
    <url>https://bitbucket.org/thibaudledent/j8583</url>
</scm>

<distributionManagement>
    <snapshotRepository>
        <id>ossrh</id>
        <url>https://oss.sonatype.org/content/repositories/snapshots</url>
    </snapshotRepository>
    <repository>
        <id>ossrh</id>
        <url>https://oss.sonatype.org/service/local/staging/deploy/maven2</url>
    </repository>
</distributionManagement>

<developers>
    <developer>
        <name>Thibaud Ledent</name>
    </developer>
</developers>
```

## Create a GPG certificate

Install `gnupg2`:

```bash
sudo apt-get install gnupg2
```

Then, you can generate a key:

```
gpg --gen-key
```


You should then see your key listed:

```
gpg2 --list-keys
```

And now distribute your public key:

```
gpg --keyserver pgpkeys.uk --send-keys DB85FB2159287141
```

The `--keyserver` parameter identifies the target key server address and `--send-keys` is the `keyid` of the key you want to distribute. You can get your `keyid` by listing the public keys.

Then, go to [http://pgpkeys.uk/](http://pgpkeys.uk/) and search `0xDB85FB2159287141`. You should see your public key (example [here](http://pgpkeys.uk/pks/lookup?search=0xDB85FB2159287141&fingerprint=on&op=index)).

If you have 'No route to host', see also this [list of available servers](https://medium.com/@xiaoxiaohou/gpg-keyserver-receive-failed-no-route-to-host-9e47a0dd83f8) (some of them might not work behind a company firewall).

## Add the necessary plugins to the pom.xml file

I use the configuration `skipRelease` to enable/disable these plugins. The list of plugins to add is listed below (example [here](https://bitbucket.org/thibaudledent/j8583/src/0bdff5d60a2aa9644025da8d0db09f9ba5ff63e2/pom.xml?at=master&fileviewer=file-view-default)):

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-site-plugin</artifactId>
            <version>3.7.1</version>
            <configuration>
                <skip>${skipRelease}</skip>
                <locales>en,es</locales>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
            <version>3.0.1</version>
            <executions>
                <execution>
                    <id>attach-sources</id>
                    <goals>
                        <goal>jar-no-fork</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-javadoc-plugin</artifactId>
            <version>3.0.1</version>
            <configuration>
                <skip>${skipRelease}</skip>
                <links>
                    <link>http://download.oracle.com/javase/8/docs/api/</link>
                    <link>http://slf4j.org/apidocs/</link>
                </links>
            </configuration>
            <executions>
                <execution>
                    <id>attach-javadocs</id>
                    <goals>
                        <goal>jar</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-gpg-plugin</artifactId>
            <version>1.6</version>
            <configuration>
                <skip>${skipRelease}</skip>
            </configuration>
            <executions>
                <execution>
                    <id>sign-artifacts</id>
                    <phase>verify</phase>
                    <goals>
                        <goal>sign</goal>
                    </goals>
                    <configuration>
                        <!-- Configuration to prevent the 'Signing Prompt' or the 'gpg: signing failed: No such file or directory' error -->
                        <!-- See https://myshittycode.com/2017/08/07/maven-gpg-plugin-prevent-signing-prompt-or-gpg-signing-failed-no-such-file-or-directory-error/ -->
                        <gpgArguments>
                            <arg>--pinentry-mode</arg>
                            <arg>loopback</arg>
                        </gpgArguments>
                    </configuration>
                </execution>
            </executions>
        </plugin>
        <plugin>
            <groupId>org.sonatype.plugins</groupId>
            <artifactId>nexus-staging-maven-plugin</artifactId>
            <version>1.6.8</version>
            <extensions>true</extensions>
            <configuration>
                <serverId>ossrh</serverId>
                <nexusUrl>https://oss.sonatype.org/</nexusUrl>
                <!-- Set this to true and the release will automatically proceed and sync to Central Repository will follow -->
                <autoReleaseAfterClose>true</autoReleaseAfterClose>
            </configuration>
        </plugin>
    </plugins>
</build>
```
## Create the Bitbucket pipeline

Now it's time to edit the [bitbucket-pipelines.yml](https://bitbucket.org/thibaudledent/j8583/src/0bdff5d60a2aa9644025da8d0db09f9ba5ff63e2/bitbucket-pipelines.yml?at=master&fileviewer=file-view-default) file to automate the Maven release. After a few round trips to StackOverflow, I managed to automatically release the library.

Here is the code of the custom pipeline, it's quite straightforward: 

```bash
image: maven:3.6.0-jdk-8-slim

pipelines:
  custom:
    release-to-maven-central:
      - step:
          script:
            - apt-get update && apt-get install -y gpg --no-install-recommends
            - export GPG_TTY=$(tty) # to fix the 'gpg: signing failed: Inappropriate ioctl for device', see https://github.com/keybase/keybase-issues/issues/2798#issue-205008630
            - echo $GPG_SECRET_KEYS | base64 --decode | gpg --batch --import # use 'batch' otherwise gpg2 is asking for a passphrase, see https://superuser.com/a/1135950
            - echo $GPG_OWNERTRUST | base64 --decode | gpg --import-ownertrust
            - mvn -V -B -s settings.xml deploy -DskipRelease=false
            # -V triggers an output of the Maven and Java versions at the beginning of the build
            # -B batch mode makes Maven less verbose
            # -s causes the usage of the local settings with the required credentials
```

In addition, I created a [settings.xml](https://bitbucket.org/thibaudledent/j8583/src/0bdff5d60a2aa9644025da8d0db09f9ba5ff63e2/settings.xml?at=master&fileviewer=file-view-default) file: 

```
<settings>
    <servers>
        <server>
            <id>ossrh</id>
            <username>${env.OSSRH_USER_TOKEN}</username>
            <password>${env.OSSRH_PWD_TOKEN}</password>
        </server>
    </servers>
    <profiles>
        <profile>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <gpg.keyname>${env.GPG_EXECUTABLE}</gpg.keyname>
                <gpg.passphrase>${env.GPG_PASSPHRASE}</gpg.passphrase>
            </properties>
        </profile>
    </profiles>
</settings>
```

For this pipeline to work, you need to set up [repository variables](https://confluence.atlassian.com/bitbucket/variables-in-pipelines-794502608.html) (in Bitbucket, go to settings -> pipelines -> repository variables):

Get the local `GPG_SECRET_KEYS`:

```
gpg -a --export-secret-keys email_address_linked_to_your_gpg_key@mail.com | base64
```

Get the `GPG_OWNERTRUST`:

```
gpg --export-ownertrust | base64
```

To get `OSSRH_USER_TOKEN`, go to [Sonatype user token](https://oss.sonatype.org/#profile;User%20Token).

I ended up with the following repository variables:

- `GPG_PASSPHRASE`
- `OSSRH_USER_TOKEN`
- `OSSRH_PWD_TOKEN`
- `GPG_SECRET_KEYS`
- `GPG_OWNERTRUST`

# Release

To release to Maven Central, go to [branches](https://bitbucket.org/thibaudledent/j8583/branches/) and run the dedicated pipeline:

![https://bitbucket.org/thibaudledent/j8583/raw/ded5f57141cf1680b5debbfe77fa84de3e8f4282/how_to_release.gif](https://bitbucket.org/thibaudledent/j8583/raw/ded5f57141cf1680b5debbfe77fa84de3e8f4282/how_to_release.gif)

After a few minutes, you will then see the artifacts in [search.maven.org](https://search.maven.org/artifact/org.bitbucket.thibaudledent.j8583/j8583/1.13.4/jar). You can use the release in a `pom.xml`:

```xml
<dependency>
  <groupId>org.bitbucket.thibaudledent.j8583</groupId>
  <artifactId>j8583</artifactId>
  <version>1.13.4</version>
</dependency>
```

# Conclusion

The part that is the most time consuming is the process around the Maven release (Jira ticket, creation of the GPG key...). Setting up the pipeline only takes a few minutes and, once done, will save you a lot of time with each release!

# References

- [https://dzone.com/articles/continuous-integration-to-maven-central-for-free](https://dzone.com/articles/continuous-integration-to-maven-central-for-free)
- [https://bitbucket.org/simpligility/ossrh-pipeline-demo/src/master/](https://bitbucket.org/simpligility/ossrh-pipeline-demo/src/master/)
- [http://mbcoder.com/publishing-to-maven-central/](http://mbcoder.com/publishing-to-maven-central/)
- [https://dzone.com/articles/publish-your-artifacts-to-maven-central](https://dzone.com/articles/publish-your-artifacts-to-maven-central)