# Instalation

## With Container
- Create Folder secure-code
  ```bash
  mkdir secure-code
  cd secure-code
  ```
- Download Script
  ```bash
  wget https://raw.githubusercontent.com/islamyakin/secure-coding/main/sonarqube/docker-compose.yaml
  ```
- Run with docker compose
  ```bash
  docker compose up -d
   ```
## Without Container

- Install OpenJDK 17
 ```bash
 sudo apt install -y openjdk-17-jdk
 ```

- Install and Configure PostgreSQL
 ```bash
 sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" /etc/apt/sources.list.d/pgdg.list'
 wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -
 sudo apt install postgresql postgresql-contrib -y
 sudo systemctl enable postgresql
 sudo systemctl enable postgresql
 ```

- Create User and Database for sonaqube
 ```bash
 sudo -i -u postgres
 psql
 CREATE USER "sonar-secure";
 ALTER USER "sonar-secure" WITH ENCRYPTED password 'S0narS3cr3T!!';
 CREATE DATABASE ddsonarsecure OWNER "sonar-secure";
 GRANT ALL PRIVILEGES ON DATABASE ddsonarsecure to "sonar-secure";
 ```

- Download sonarqube from source
  https://www.sonarsource.com/products/sonarqube/downloads/success-download-community-edition/

- Extract sonarqube and copy to /opt/
 ```bash
 unzip sonarqube-10.6.0.92116.zip
 sudo mv sonarqube-10.6.0.92116 /opt/sonarqube
 ```

- Create User and group for sonarqube
 ```bash
 sudo groupadd sonargrp
 sudo useradd -d /opt/sonarqube -g sonargrp sonargrp
 sudo chown -R sonargrp:sonargrp /opt/sonarqube
 ```

- Configure sonarqube
```bash
sudo nano /opt/sonarqube/conf/sonar.properties
```


Add the database user and Database password you created
```bash
sonar.jdbc.username=sonar-secure
sonar.jdbc.password=S0narS3cr3T!!
sonar.jdbc.url=jdbc:postgresql://localhost:5432/ddsonarsecure
```

- Edit the sonar script file
```bash
sudo nano /opt/sonarqube/bin/linux-x86-64/sonar.sh
```
Add the following line
```bash
RUN_AS_USER=sonargrp
```

- Setup Systemd service
```bash
sudo nano /etc/systemd/system/sonar.service
```

Paste the following lines to the file.
```bash
[Unit]
Description=SonarQube service
After=syslog.target network.target
[Service]
Type=forking
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
User=sonargrp
Group=sonargrp
Restart=always
LimitNOFILE=65536
LimitNPROC=4096
[Install]
WantedBy=multi-user.target
```

- Enable the SonarQube service to run at system startup.
```bash
sudo systemctl daemon-reload
sudo systemctl enable sonar
sudo systemctl start sonar
sudo systemctl status sonar
```

