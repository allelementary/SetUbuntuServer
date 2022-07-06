# Set Ubuntu Server

Guide for hosting an application on Ubuntu server

## Install & Update Packages

#### Login into Ubuntu server using provided IP address

```bash
ssh root@127.0.0.0
```

#### Update system packages

```bash
sudo apt update && sudo apt upgrade -y
```

#### Check if python is installed

```commandline
python3 --version
```

#### Install python, pip, virtualenv

```bash
sudo apt install python

sudo apt install python3-pip -y

sudo pip3 install virtualenv
```

## Install PostgreSQL

```commandline
sudo apt install postgresql postgresql-contrib -y
```

#### list users on the Ubuntu machine

```commandline
cat /etc/passwd
```

#### Change user to get access to psql and change password to `postgres`

```bash
su - postgres

psql -U postgres

\password postgres

\q
```

#### Edit postgresql.conf

```commandline
cd /etc/postgresql/12/main/

sudo vi postgresql.conf
```

```bash
# postgresql.conf
"Connections & authentication"
add listen_addresses = '*'
```

#### Edit pg_hba.conf

```commandline
sudo vi pg_hba.conf
```

```bash
# pg_hba.conf
postgres -> md5
all -> md5
IPv4 (address) -> 0.0.0.0/0
IPv6 (address) -> ::/0
```

#### Restart PostgreSQL

```commandline
systemctl restart postgresql
```

## Add User

#### Create user `admin`

```bash
adduser admin

su - admin
```

#### Change user back to `root` to give rights to `admin`

```commandline
exit

usermod -aG sudo admin
```

#### Exit server and login under `admin`

```bash
ssh admin@0.0.0.0
```

## Upload App & Set-Up Environment

```bash
mkdir app

cd app

virtualenv venv

mkdir src

cd src

git clone {repository} .

sudo apt install libpq-dev

sudo apt-get install python3-dev
```

#### Activate venv and install requirements

```commandline
source venv/bin/activate

pip install -r requirements.txt
```

#### Set Environment Variables

```bash
cd ~/ 

touch .env

vi .env
```

#### Add variables

Add database credentials and other sensitive data

```bash
# example .env
DATABASE_HOSTNAME=localhost
DATABASE_PORT=5432
DATABASE_PASSWORD=postgres
DATABASE_NAME=my_base
DATABASE_USERNAME=postgres
SECRET_KEY=secret_key
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
```

#### Activate environment

```bash
source .env

set -o allexport; source /home/admin/.env; set +o allexport

vi .profile
```

#### Add at the end of .profile:

```bash
set -o allexport; source /home/admin/.env; set +o allexport
```

#### Create DataBase & Make Migrations

## Set-Up Gunicorn in a Background

```bash
pip install gunicorn

gunicorn -w 1 -k uvicorn.workers.UvicornWorker app.main:app --bind 0.0.0.0:8000

ps -aef | grep -i gunicorn

check workers
```

#### Start process in a background

##### Create service

```bash
cd /etc/systemd/system

create gunicorn.service

sudo vi api.service

systemctl start api

systemctl status api

sudo systemctl enable api
```

## Set Nginx

#### Install nginx

```bash
sudo apt install nginx -y
```

#### Create default

```bash
cd /etc/nginx/sites-available/

sudo vi default
```

```bash
# default
server {
        listen 80 ;
        listen [::]:80 ;

        server_name _;

        location / {
                proxy_pass http://localhost:8000;
                proxy_http_version 1.1;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $http_host;
                proxy_set_header X-NginX-Proxy true;
                proxy_redirect off;
        }
}
```

#### Start nginx & install

```bash
sudo systemctl start nginx (journalctl -xe) --logs

sudo apt install snapd -y

sudo snap install core; sudo snap refresh core
```

#### Set certbot


```bash
sudo snap install --classic certbot

sudo certbot --nginx

# provide domains domain.com www.domain.com
```
