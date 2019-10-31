## Continuous Deploy with Infrastructure as code

---
@title[Background]
@snap[north heading]
Background Reading
@snapend
[https://fenixdb.atlassian.net/wiki/spaces/SOF/pages/417824996/FenixDB+Continuous+Deployment+Pipeline+W.I.P](https://fenixdb.atlassian.net/wiki/spaces/SOF/pages/417824996/FenixDB+Continuous+Deployment+Pipeline+W.I.P)

---
@title[Appspec]
@snap[north heading text-pink]
`appspec.yml`
@snapend

```
version: 0.0
os: linux
files:
  - ...
permissions:
  - ...
hooks:
  ApplicationStop:
    - ...
  BeforeInstall:
    - ...
  AfterInstall:
    - ...
  ApplicationStart:
    - ...
  ValidateService:
    - ...
```

---
@snap[north-east span-100 text-pink text-06]
`appspec.yml`
@snapend

```yaml zoom-18
version: 0.0
os: linux
files:
  - source: /
    destination: /opt/fenixdb
permissions:
  - object: /opt/fenixdb
    owner: www-data
    group: www-data
    #mode: 777
    type:
      - directory
      - file
  - object: /opt/fenixdb_git_commit
    owner: root
    group: www-data
    type:
      - file
  - object: /opt/fenixdb_queue_name
    owner: root
    group: www-data
    type:
      - file
  - object: /etc/supervisor/conf.d/celery.conf
    owner: root
    group: www-data
    type:
      - file
hooks:
  ApplicationStop:
    - location: scripts/codedeploy/stop_services
      timeout: 300
      runas: root
  BeforeInstall:
    - location: scripts/codedeploy/create_pid_dir
      timeout: 3
      runas: root
    - location: scripts/codedeploy/extract_commit
      timeout: 3
      runas: root
  AfterInstall:
    - location: scripts/codedeploy/setup_git
      runas: root
    - location: scripts/codedeploy/symlink_workers
      runas: root
    - location: scripts/codedeploy/install_dependencies
      runsas: root
    - location: scripts/codedeploy/install_requirements
      runas: root
  ApplicationStart:
    - location: scripts/codedeploy/start_services
      timeout: 30
      runas: root
  ValidateService:
    - location: scripts/codedeploy/status_check
      timeout: 30
      runas: root
```

@snap[south span-100 text-gray text-08]
@[3-5](Specify where to copy files on host)
@[6-13, zoom-13](Define permissions for files and dirs)
@[14,19,24, zoom-12](Stage critical files not part of repo)
@[29-33, zoom-13](Stop all services)
@[34,35,38, zoom-12](Prep the environment)
@[41,42,44,46,48, zoom-12](Setup the application)
@[50,51, zoom-12](Start the application)
@[54,55, zoom-12](Validate all services are running as expected)
@snapend

---
@snap[north-east span-100 text-pink text-06]
`scripts/codedeploy/stop_services`
@snapend

```
#!/bin/bash
sudo service uwsgi stop
sudo supervisorctl stop all
```

---
@snap[north-east span-100 text-pink text-06]
`scripts/codedeploy/create_pid_dir`
@snapend

```
#!/bin/bash
mkdir /var/run/fenixdb
chown -R www-data:www-data /var/run/fenixdb
```
