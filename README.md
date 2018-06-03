# nexus-oss-npm-proxy

## Start Server
### Using Openshift
oc new-app sonatype/nexus:oss

### Using single Docker container 
```
docker run -d -p 8081:8081 --name nexus sonatype/nexus:oss
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

Create a new Repository -> Proxy Repository -> npm -> Fill the form
input registry for npm https://registry.npmjs.org

Get the Repository path from UI (similar to http://localhost:8081/nexus/content/repositories/npm-all)

### Validate

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

##### References 
* https://help.sonatype.com/repomanager3/node-packaged-modules-and-npm-registries
* https://help.sonatype.com/repomanager3/quick-start-guide---proxying-maven-and-npm
* https://www.youtube.com/watch?v=6Y5sRcY92d0 
