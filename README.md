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

Create folders for artifactory database:
```
sudo mkdir /artifactory
```
```
sudo mkdir /artifactory/access
```
```
sudo mkdir /artifactory/db
```
```
sudo mkdir /artifactory/data
```

Create folder for logs:
```
sudo mkdir /var/log/artifactory
```

Create folder for backups:
```
sudo mkdir /var/backups/artifactory
```

Grant permit to all folders:
```
sudo chown -R artifactory:artifactory /artifactory
```
```
sudo chown artifactory:artifactory /var/log/artifactory
```
```
sudo chown artifactory:artifactory /var/backups/artifactory
```

Copy **etc/init.d/artifactory** to **/etc/init.d** folder:
```
sudo cp ./etc/init.d/artifactory /etc/init.d
```

Copy **usr/sbin/artifactory** to **/usr/sbin** folder:
```
sudo cp ./usr/sbin/artifactory /usr/sbin
```

Copy **usr/bin/artutil** to **/usr/bin** folder:
```
sudo cp ./usr/bin/artutil /usr/bin
```

Make all files executable:
```
sudo chmod a+x /etc/init.d/artifactory
```
```
sudo chmod a+x /usr/sbin/artifactory
```
```
sudo chmod a+x /usr/bin/artutil
```

Register service:
```
sudo update-rc.d artifactory defaults
```

Specify database root password in **/usr/sbin/artifactory** file:
```
docker run ... -e DB_ROOT_PASSWORD="<some_password>" ...
```

Specify artifactory database user password in **/usr/sbin/artifactory** file:
```
docker run ... -e DB_USER_PASSWORD="<some_password>" ...  
```

Start artifactory service:
```
sudo service artifactory start
```

Initialize artifactory database:
```
sudo artutil initialize
```

Management
----------
### Service management
```
sudo service artifactory (start|stop|status|restart)
```
