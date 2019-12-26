# Artifactory web server
Docker image for artifactory web server.

Container runs as non-root user.
This user owns artifactory process and owns artifactory database.

To run container next volumes should be mapped:
* folder for SQL database
* folder for artifactory data
* folder for artifactory access
* log folder
* backup folder

## Installation
### Installation from docker image
1. Pull docker image.

2. Create user and group to own artifactory files and to run docker container:
    ```
    sudo groupadd -g 966 artifactory
    ```
    ```
    useradd -u 966 -g 966 -M artifactory
    ```

3. Proceed to configuration.

### Installation from source
1. Pull project sources from version control system.

2. Create user and group to own artifactory files and to run docker container:
    ```
    sudo useradd -r artifactory
    ```

3. Make **build** executable:
    ```
    sudo chmod u+x ./build
    ```

4. Execute **build**:
    ```
    sudo ./build artifactory
    ```

5. Proceed to configuration.

### Configuration
1. Create folders for artifactory database:
    ```
    sudo mkdir /artifactory
    ```
    ```
    sudo mkdir /artifactory/db
    ```
    ```
    sudo mkdir /artifactory/data
    ```
    ```
    sudo mkdir /artifactory/accessd
    ```
    ```
    sudo mkdir /artifactory/accesse
    ```

2. Create folder for logs:
    ```
    sudo mkdir /var/log/artifactory
    ```

3. Create folder for backups:
    ```
    sudo mkdir /var/backups/artifactory
    ```

4. Grant permit to all folders:
    ```
    sudo chown -R artifactory:artifactory /artifactory
    ```
    ```
    sudo chown artifactory:artifactory /var/log/artifactory
    ```
    ```
    sudo chown artifactory:artifactory /var/backups/artifactory
    ```

5. Copy **etc/init.d/artifactory** to **/etc/init.d** folder:
    ```
    sudo cp ./etc/init.d/artifactory /etc/init.d
    ```

6. Copy **usr/sbin/artifactory** to **/usr/sbin** folder:
    ```
    sudo cp ./usr/sbin/artifactory /usr/sbin
    ```

7. Copy **usr/bin/artutil** to **/usr/bin** folder:
    ```
    sudo cp ./usr/bin/artutil /usr/bin
    ```

8. Make all files executable:
    ```
    sudo chmod a+x /etc/init.d/artifactory
    ```
    ```
    sudo chmod a+x /usr/sbin/artifactory
    ```
    ```
    sudo chmod a+x /usr/bin/artutil
    ```

9. Register service:
    ```
    sudo update-rc.d artifactory defaults
    ```

10. Specify database root password in **/usr/sbin/artifactory** file:
    ```
    docker run ... -e DB_ROOT_PASSWORD="<some_password>" ...
    ```

11. Specify artifactory database user password in **/usr/sbin/artifactory** file:
    ```
    docker run ... -e DB_USER_PASSWORD="<some_password>" ...  
    ```

12. Start artifactory service:
    ```
    sudo service artifactory start
    ```

13. Initialize artifactory database:
    ```
    sudo artutil initialize
    ```

14. Restart artifactory service:
    ```
    sudo service artifactory restart
    ```

## Management
### Service management
```
sudo service artifactory (start|stop|status|restart)
```

### Create backup
```
sudo artutil backup <filename>
```

Backup file **/var/backups/artifactory/&lt;filename&gt;.tar.gz** will be created.

### Restore backup
```
sudo artutil restore <filename>
```

### Command line (bash)
```
sudo artutil bash
```

## Apache mod_proxy configuration
Artifactory web server can be located with another web applications.
For example, mercurial, bugzilla, wiki etc can be run as docker containers on the same host.
In this case apache server can be used to redirect requests to different docker containers.

1. Enable apache mod_proxy:
    ```
    sudo a2enmod proxy proxy_ajp proxy_html proxy_http rewrite deflate headers proxy_balancer proxy_connect
    ```

2. Configure proxy:
    ```
    <VirtualHost *:80>
        ...
        ProxyPreserveHost On
        <Proxy *>
            Order allow,deny
            Allow from all
        </Proxy>
        ...
    </VirtualHost>
    ```

3. Copy **./etc/apache2/sites-available/access.conf** to **/etc/apache2/sites-available** folder:
    ```
    sudo cp ./etc/apache2/sites-available/access.conf /etc/apache2/sites-available
    ```

4. Copy **./etc/apache2/sites-available/artifactory.conf** to **/etc/apache2/sites-available** folder:
    ```
    sudo cp ./etc/apache2/sites-available/artifactory.conf /etc/apache2/sites-available
    ```

5. Enable apache artifactory sites:
    ```
    sudo a2ensite access
    ```
    ```
    sudo a2ensite artifactory
    ```

6. Restart apache service:
    ```
    sudo service apache2 restart
    ```

## HOW TO
### How to change database root password
Stop artifactory service:
```
sudo service artifactory stop
```

Specify new database root password in **/usr/sbin/artifactory** file:
```
docker run ... -e DB_ROOT_PASSWORD="<new_password>" ...
```

Start artifactory service:
```
sudo service artifactory start
```

Run the following command:
```
sudo artutil changeRootPassword "<old_password>"
```

### How to change artifactory database user password
Stop artifactory service:
```
sudo service artifactory stop
```

Specify new artifactory database user password in **/usr/sbin/artifactory** file:
```
docker run ... -e DB_USER_PASSWORD="<new_password>" ...  
```

Start artifactory service:
```
sudo service artifactory start
```

Run the following command
```
sudo artutil changeUserPassword
```

### How to specify special characters in password
Special characters should be escaped:
```
docker run ... -e DB_ROOT_PASSWORD="pa\$\$word" ...
```
```
docker run ... -e DB_USER_PASSWORD="pas\$11" ...  
```

### How to create cron job for backups
```
sudo crontab -l | { cat; echo "minute hour * * * /usr/bin/artutil backup <filename>"; echo ""; } | sudo crontab -
```

# Donation
If you find my code useful, you can [bye me a coffee](https://www.paypal.me/dshapovalov)
