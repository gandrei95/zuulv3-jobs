# Official Documentation
- zuul docs: https://zuul-ci.org/docs/zuul/index.html
- nodepool docs: https://zuul-ci.org/docs/nodepool/

OS Required: Ubuntu 18.0x
Why? Because of `bubblewrap` package which is available only on Ubuntu 18.0x.

```bash
$ apt-get update -y && apt-get upgrade -y
$ apt-get install git -y
$ apt-get install python3 python3-dev python3-pip -y
```

# Install Nodepool

1. Install apache-zookeeper [nodepool requirement]
```bash
$ apt-get install -y zookeeper zookeeperd
```
2. Clone nodepool repository and prepare directories and files
```bash
$ git clone https://github.com/openstack-infra/nodepool

$ groupadd nodepool
$ sudo useradd nodepool --shell /bin/bash --home-dir /var/lib/nodepool --create-home -g nodepool
$ mkdir /etc/nodepool/
$ mkdir /var/log/nodepool
$ chown -R nodepool:nodepool /var/log/nodepool
$ chown -R nodepool:nodepool /etc/nodepool
$ chmod 775 /var/log/nodepool

$ pushd nodepool/
$ pip3 install .

$ touch /etc/systemd/system/nodepool-launcher.service
```
```bash
$ su nodepool
$ touch /etc/nodepool/nodepool.yaml
$ touch /etc/nodepool/launcher-logging.conf
## copy the nodepool.yaml file content
$ mkdir -p /var/lib/nodepool/.config/openstack
$ touch clouds.yaml ## contains the definition of the clouds used
$ sudo systemctl start nodepool-launcher
$ sudo systemctl enable nodepool-launcher
```

```bash
## useful commands for nodepool; Must be executed using nodepool user
$ nodepool --help
# ------
	list                list nodes
    image-list          list images from providers
    dib-image-list      list images built with diskimage-builder
    image-build         build image using diskimage-builder
    alien-image-list    list images not accounted for by nodepool
    delete              place a node in the DELETE state
    image-delete        delete an image
    dib-image-delete    Delete a dib built image from disk along with all
                        cloud uploads of this image
    config-validate     Validate configuration file
    request-list        list the current node requests
    info                Show provider data from zookeeper
    erase               Erase provider data from zookeeper
```


# Install Zuul

1. Install mysql server
```bash
$ sudo apt-get install mysql-server mysql-client -y
$ pip3 install mysqlclient
```
2. Create the database where build results are stored
```SQL
CREATE USER 'zuulci'@'%' IDENTIFIED BY 'zuulci';
CREATE DATABASE buildsdb;
GRANT ALL PRIVILEGES ON buildsdb.* to 'zuulci'@'%';
FLUSH PRIVILEGES;
```

3. Clone zuul repository
```bash
$ git clone https://github.com/openstack-infra/zuul
$ pushd zuul/

## this step is required in order to prepare the files used by zuul-web service for the web interface
$ ./tools/install-js-tools.sh
$ yarn install
$ yarn build

$ sudo groupadd zuul
$ sudo useradd zuul --home-dir /var/lib/zuul --create-home -g zuul
$ sudo mkdir /etc/zuul/
$ sudo mkdir /var/log/zuul/
$ sudo chown zuul:zuul /var/log/zuul/
$ sudo mkdir /var/lib/zuul/builds
$ sudo chown zuul:zuul /var/lib/zuul/builds
$ sudo mkdir /var/lib/zuul/.ssh
$ sudo chmod 0700 /var/lib/zuul/.ssh
$ sudo chown -R zuul:zuul /var/lib/zuul/.ssh

$ pip3 install .

$ su zuul
## create zuul.conf file
$ touch /etc/zuul/zuul.conf
## obs. private_key_file=/var/lib/zuul/.ssh/default.pem
## is the key used by zuul executors to ssh in instances

$ touch /etc/systemd/system/zuul-scheduler.service
$ touch /etc/systemd/system/zuul-executor.service
$ touch /etc/systemd/system/zuul-web.service
$ touch /etc/systemd/system/zuul-fingergw.service
```

