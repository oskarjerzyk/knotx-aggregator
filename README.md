# Development process
This section describes tools that help you with coding. We provide scripts that allows to
clone all repositories and install artifacts to Maven local / Maven Central Snapshot repositories.

## Prerequisites on Windows
Install and configure [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10) (WSL) and configure:
- Java 8 on WSL
- Gradle 5+
- Maven

## Clone all repositories
Check a `development/pull-all.sh` script to clone all Knot.x repositories. Please check `-h` option
for help. Execute this script from `knotx-aggregator/development` directory, executing it from another 
directory will fail. The `-r` option specifies directory where all Knot.x repositories 
are cloned such as `sh pull-all.sh -r ../../knotx` make sure, that provided directory exists. 

## Build Stack
Check a `development/build-stack.sh` script to build all cloned repositories. Please check `-h` option
for help. Execute this command from `knotx-aggregator/development` directory.
Specify the `-r` parameter to point the directory with cloned repositories, for example
`sh buils-stack.sh -r ../../knotx` make sure that in the provided directory repositories are properly
cloned.

Please note that the `knotx-stack` repository contains
[Gradle composite build](https://docs.gradle.org/current/userguide/composite_builds.html) definition.
The repository allows to re-build all Knot.x modules and use them during integration tests, bypassing the
need to publish artifacts to the maven repository first.

The `build-all.sh` command deploys the [Knot.x BOM file](https://github.com/Knotx/knotx-dependencies) 
(Bill Of Materials) to the local Maven repository first. The BOM file specifies external dependencies, such as used Vert.x 
version etc. Then all modules are rebuilt. After the execution, please refresh your IDE.

## Run Stack
From [here](https://github.com/Knotx/knotx-example-project) you can download example Knot.x projects.
Let's use `getting-started` project which is contained in this repository as an example.
Navigate to the project directory and build it 
```
gradlew clean build
``` 
After successful build you'll find zip archive `knotx-stack-2.0.0-SNAPSHOT.zip` under `./build/distributions`. Take into 
account that, `knotx-stack` version may be different in the future. Unzip it and navigate to `knotx` directory. 
From there execute command 
```
chmod +x bin/knotx
bin/knotx run-knotx
```
To check if this example projects was built properly, just open [link](http://localhost:8092/content/books.html)
listed in the repository. If you will see properly rendered page without errors it means that everything with Knot.x 
is up and running

## Build Stack & Docker & Starter-Kit

// TODO

## Use cases
After cloning the repository on Unix please change permissions:

```
$>git clone git@github.com:Knotx/knotx-aggregator.git
$>chmod -R 755 knotx-aggregator/**/*.sh
```

### Install my changes to M2 repository
From `development` directory run:
```
$>./pull-all.sh -r projects/knotx -b feature/my-changes -m origin/master
$>./build-stack.sh -r projects/knotx
```

### Deploy all to Maven Central SNAPSHOT repository
From `development` directory run:
```
$>./pull-all.sh -r projects/knotx -b master -f
$>./build-stack.sh -r projects/knotx -d
```

# Release process
This section describes release process.

## Prerequisites
1. Sonatype.org [JIRA](https://issues.sonatype.org/secure/Signup!default.jspa) account

2. Your Sonatype.org account needs to be added to the Knot.x project (if it isn't, please contact the Knot.x team
via [User Group](https://groups.google.com/forum/#!forum/knotx) or [Gitter Chat](https://gitter.im/Knotx/Lobby).

3. A GPG key generated for the email you have registered on the Sonatype.org JIRA
(Follow the [Working with PGP Signatures](http://central.sonatype.org/pages/working-with-pgp-signatures.html)
guide to get one).
**Don't forget to deploy your public key to the key server!**

4. Add a `<server>` entry to [your `settings.xml` file](https://maven.apache.org/settings.html#Introduction)
   ```xml
   <servers>
     ...
     <server>
       <id>ossrh</id>
       <username>your_sonatype_org_jira_username</username>
       <password>your_sonatype_org_jira_password</password>
     </server>
       ...
   </servers>    
   ```

5. Assuming you have created account on [Docker Hub](https://hub.docker.com/) and you're assigned to the Knot.x organization, add server entry to your `settings.xml` to enable pushing to the docker registry.
```xml
	<server>
		<id>registry.hub.docker.com</id>
		<username>[Username]</username>
		<password>[password]</password>
	</server>
```

6. Bintray account created (to release Knot.x binary distributions) and token generated.

7. Update your `gradle.properties` with
```
signing.keyId=24875D73
signing.password=secret
signing.secretKeyRingFile=/Users/me/.gnupg/secring.gpg
```
and
```
ossrhUsername=your_sonatype_org_jira_username
ossrhPassword=your_sonatype_org_jira_password
```
For releasing gradle repos.
Also add
```
org.gradle.internal.http.connectionTimeout=60000
org.gradle.internal.http.socketTimeout=300000
```
To fix the read timeouts gradle issue.

## Procedure
> Prerequisites
> Install [Mac OSX/Gsed](http://gridlab-d.shoutwiki.com/wiki/Mac_OSX/Gsed)

From the repository `release` directory execute following commands:

1. Clone repos. (It will do a shallow clone of all repos being a subject of the release procedure)
```bash
./clone.sh
```

2. Start release & provide the release version number
```bash
./start-release.sh 2.0.0
```

3. Validate release on staging repos

4. Release docker image:
```bash
./release-docker.sh
```

5. Close the release
Create manually new versions for `examples` and `distro` packages at bintray.
Run:
```bash
./close-release.sh 1.5.0 1.5.1-SNAPSHOT <bintrayUser> <bintrayToken>
```

6. Manual actions:
  - close and promote repos released by gradle manually at oss.sonatype.org
  - publish bintray files
  - update website - binary distro links
  - update wiki
  - publish release blog

## ToDo
- automatically close and promote repos released by gradle manually at oss.sonatype.org
- update `knotx-website`
  - binary distro links
  - `index` version
  - produce `blog/release-VERSION/` page that consists of all `CHANGELOG.md` changes for every module
  - deploy `SNAPSHOT` versions after release to make development of new versions possible and satisfy failing travis
