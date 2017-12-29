# ssb-pub

easily host your own [Secure ScuttleButt (SSB)](https://www.scuttlebutt.nz) pub in a docker container

to run a pub you need to have a static public IP, ideally with a DNS record (i.e.`<hostname.yourdomain.tld>`)

if you feel like sharing your pub, please add it to [the informal registry of pubs](https://github.com/ssbc/scuttlebot/wiki/Pub-Servers) as request-only or with a re-usable invite (`invite.create 1000`)!

## easy setup

[![Install on DigitalOcean](http://butt.nz/button.svg)](http://butt.nz)

wait some time, then once your server is ready

```shell
ssh root@your.ip.address.here
```

check if your server is up

```shell
./sbot whoami
```

create your first invite!

```shell
./sbot invite.create 1
```

## advanced setup

on a fresh Debian 9 box, as root

```shell
apt update
apt upgrade -y
apt install -y apt-transport-https ca-certificates curl software-properties-common
wget https://download.docker.com/linux/debian/gpg -O docker-gpg
sudo apt-key add docker-gpg
echo "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee -a /etc/apt/sources.list.d/docker.list
apt update
apt install -y docker-ce
systemctl start docker
systemctl enable docker
```

### install image

#### (option a) pull image from docker hub

```shell
docker pull ahdinosaur/ssb-pub
```

#### (option b) build image from source

from GitHub:

```shell
git clone https://github.com/ahdinosaur/ssb-pub.git
cd ssb-pub
docker build -t ssb-pub .
```

### start service

#### step 1. create a directory on the docker host for persisting the pub's data

```shell
mkdir /root/ssb-pub-data
chown -R 1000:1000 /root/ssb-pub-data
```

> if migrating from an old server, copy your old `secret` and `gossip.json` (maybe also `blobs`) now.
>
> ```
> rsync -avz /root/ssb-pub-data/blobs/sha256/ $HOST:/root/ssb-pub-data/blobs/sha256/
> ```

#### step 2. run the container

```shell
ssb_host=<hostname.yourdomain.tld>

docker run -d --name sbot \
   -v /root/ssb-pub-data/:/home/node/.ssb/ \
   -e ssb_host="$ssb_host" \
   -p 8008:8008 --restart unless-stopped \
   ahdinosaur/ssb-pub
```

### create invites

from your remote machine

```shell
ssb_host=<hostname.yourdomain.tld>

docker run -it --rm \
   -v /root/ssb-pub-data/:/home/node/.ssb/ \
   -e ssb_host="$ssb_host" \
   ahdinosaur/ssb-pub \
   invite.create 1
```

from your local machine, using ssh

```shell
ssb_host=<hostname.yourdomain.tld>

ssh root@$ssb_host \
  docker run --rm \
     -v /root/ssb-pub-data/:/home/node/.ssb/ \
     -e ssb_host="$ssb_host" \
     ahdinosaur/ssb-pub \
     invite.create 1
```

### control service

- `docker stop sbot`
- `docker start sbot`
- `docker restart sbot`

### setup auto-healer

using [somarat/healer](https://github.com/somarat/healer)

```shell
docker pull ahdinosaur/healer
```

```shell
docker run -d --name healer \
  -v /var/run/docker.sock:/tmp/docker.sock \
  --restart unless-stopped \
  ahdinosaur/healer
```
