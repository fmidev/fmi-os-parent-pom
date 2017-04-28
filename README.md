# Parent POM for FMI Open Source Maven projects

## FMI OS Maven artifact repositories

Artifacts from the FMI OS Maven projects are publised in two repositories hosted at Github:
* [fmi-os-mvn-repo](https://github.com/fmidev/fmi-os-mvn-repo/) for release versions, and
* [fmi-os-mvn-snapshot](https://github.com/fmidev/fmi-os-mvn-snapshot-repo/) for shared snapshot versions.

To include artifacts from these repositories in your projects, please add the following either to the pom.xml of the depending project(s), or in your settings.xml:

```xml
<repositories>
  <repository>
    <id>fmi-os-mvn-snapshot-repo</id>
    <url>https://raw.githubusercontent.com/fmidev/fmi-os-mvn-snapshot-repo/master</url>
    <snapshots>
      <enabled>true</enabled>
      <updatePolicy>always</updatePolicy>
    </snapshots>
    <releases>
      <enabled>false</enabled>
    </releases>
  </repository>
  <repository>
    <id>fmi-os-mvn-release-repo</id>
    <url>https://raw.githubusercontent.com/fmidev/fmi-os-mvn-repo/master</url>
    <snapshots>
      <enabled>false</enabled>
    </snapshots>
    <releases>
      <enabled>true</enabled>
      <updatePolicy>daily</updatePolicy>
    </releases>
  </repository>
 </repositories>
```

Note that the repositories above need to use the "raw" Github address and not the HTML versions under https://github.com/fmidev/

## Releasing/deploying artifacts into the FMI OS Maven artifact repositories

The settings included in this parent pom enable deploying snapshot and release artifacts into the FMI OS Maven repositories. This mechanism uses the two-phase deploy method as described in http://stackoverflow.com/questions/14013644/hosting-a-maven-repository-on-github:

To deploy the released artifact into the fmi-os-mvn-repo during the release, enable the profile "fmi-os-release":


    mvn clean release:prepare release:perform -Pfmi-os-release

To deploy a snapshot artifact into the fmi-os-mvn-snapshot-repo, enable the profile "fmi-os-snapshot":

    mvn clean deploy -Pfmi-os-snapshot

Anyone with modify permissions for these Github repositories is able to deploy to them.

### Technical details for deployment settings

First, the artifacts are deployed into a local directory:
```xml
  <distributionManagement>
    <repository>
      <id>internal.repo</id>
      <name>Temporary Staging Repository</name>
      <url>file://${project.build.directory}/mvn-repo</url>
    </repository>
  </distributionManagement>
```

Then the Github site Maven plugin is used to package and transfer the deployed artifacts into the Maven repository hosted at Github:

```xml
<profile>
  <id>fmi-os-snapshot</id>
  <build>
    <pluginManagement>
      <plugins>
		 <plugin>
		    <groupId>com.github.github</groupId>
		    <artifactId>site-maven-plugin</artifactId>
		    <version>${site-maven-plugin.version}</version>
		    <configuration>
		      <message>Maven artifacts for ${project.version}</message>  <!-- git commit message -->
		      <noJekyll>true</noJekyll>                                  <!-- disable webpage processing -->
		      <outputDirectory>${project.build.directory}/mvn-repo</outputDirectory> <!-- matches distribution management repository url above -->
		      <branch>refs/heads/master</branch>                       <!-- remote branch name -->
		      <includes>
		        <include>**/*</include>
		      </includes>
		      <repositoryName>${github-mvn-snapshot-repo}</repositoryName>      <!-- github repo name -->
		      <repositoryOwner>${github-repo-owner}</repositoryOwner>    <!-- github username  -->
		      <merge>true</merge>
		    </configuration>
		    <executions>
		      <!-- run site-maven-plugin's 'site' target as part of the build's normal 'deploy' phase -->
		      <execution>
		        <goals>
		          <goal>site</goal>
		        </goals>
		        <phase>deploy</phase>
		      </execution>
		    </executions>
		  </plugin>
	  </plugins>
    </pluginManagement>
  </build>
</profile> 
```

At the deploy phase of the Maven lifecycle, the submitted contents are is merged with the existing contents of the master branch or the release or snapshot Github repositories. This is important, because otherwise each deployment would remove previously deployed artifacts from other projects.



