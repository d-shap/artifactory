Artifactory web server
======================
Docker image for artifactory web server.

Container runs as non-root user.
This user owns artifactory process and owns artifactory database.

To run container next volumes should be mapped
* folder for SQL database
* folder for artifactory data
* log folder
* backup folder

Installation
------------
Create user and group to own artifactory files and to run docker container:
```
sudo useradd -r artifactory
```

Make **build** executable:
```
sudo chmod u+x ./build
```

Execute **build**:
```
sudo ./build artifactory
```
