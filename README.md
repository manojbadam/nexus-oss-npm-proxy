# nexus-oss-npm-proxy
Nexus Registry can be used to setup as proxy to the npm/maven registry and reduce more downloads from public internet

## Start Server
### Using Openshift
oc new-app sonatype/nexus:oss

### Using single Docker container 
```
docker run -d -p 8081:8081 --name nexus sonatype/nexus:oss
(or)
docker run -d -p 8081:8081 --name nexus -v /some/dir/nexus-data:/sonatype-work sonatype/nexus
```
### Using Two Docker containers for persistent storage
```
docker run -d --name nexus-data sonatype/nexus:oss echo "data-only container for Nexus"
docker run -d -p 8081:8081 --name nexus --volumes-from nexus-data sonatype/nexus:oss
```

wait for atleast 10mins for the registry to initialize and hit this endpoint 
```
curl http://localhost:8081/nexus/service/local/status
```
### Server Setup
After the health endpoint is active, try accessing the UI 
http://localhost:8081/nexus

Login using the default credentials `admin/admin123`

### NPM Server Setup
Create a new Repository -> Proxy Repository -> npm -> Fill the form
input registry for npm https://registry.npmjs.org

Get the Repository path from UI (similar to http://localhost:8081/nexus/content/repositories/npm-all)

#### Validate

Goto any node project directory 
##### create .npmrc file with following contents
```
registry = http://localhost:8081/nexus/content/repositories/badam/
```
##### Run a sample npm install command 
```
npm install grunt
```
##### Check the time of duration 
```
time npm install grunt
```
##### Check if the local npm registry is hitting our proxy server
```
npm --loglevel info install grunt
```

check this curl http://localhost:8081/nexus/content/repositories/badam/ to find out cached repositories

### Maven Server Setup
Create a new Repository -> Proxy Repository -> maven -> Fill the form
input registry for maven http://repo.maven.apache.org/maven2/

Get the Repository path from UI (similar to http://localhost:8081/nexus/content/repositories/mvn-all/)

#### Validate 

update mvn settings.xml (~/.m2/settings.xml)
```
<settings>
  <mirrors>
	<mirror>
  	<!--This sends everything else to /public -->
  	<id>nexus</id>
  	<mirrorOf>*</mirrorOf>
  	<url>http://localhost:8081/nexus/content/repositories/mvn-all/</url>
	</mirror>
  </mirrors>
  <profiles>
	<profile>
  	<id>nexus</id>
  	<!--Enable snapshots for the built in central repo to direct -->
  	<!--all requests to nexus via the mirror -->
  	<repositories>
    	<repository>
      	<id>central</id>
      	<url>http://central</url>
      	<releases><enabled>true</enabled></releases>
      	<snapshots><enabled>true</enabled></snapshots>
    	</repository>
  	</repositories>
 	<pluginRepositories>
    	<pluginRepository>
      	<id>central</id>
      	<url>http://central</url>
      	<releases><enabled>true</enabled></releases>
      	<snapshots><enabled>true</enabled></snapshots>
    	</pluginRepository>
  	</pluginRepositories>
	</profile>
  </profiles>
  <activeProfiles>
	<!--make the profile active all the time -->
	<activeProfile>nexus</activeProfile>
  </activeProfiles>
</settings>
```

##### References 
* https://help.sonatype.com/repomanager3/node-packaged-modules-and-npm-registries
* https://help.sonatype.com/repomanager3/quick-start-guide---proxying-maven-and-npm
* https://www.youtube.com/watch?v=6Y5sRcY92d0 
* https://maven.apache.org/guides/mini/guide-mirror-settings.html
* https://books.sonatype.com/mcookbook/reference/repoman-sect-proxy-repo.html
