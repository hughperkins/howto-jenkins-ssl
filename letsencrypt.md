# Let's encrypt

Getting a standard certificate for your jenkins, using Let's Encrypt.

By comparison with the self-signed, the pre-requisites are significantly stricter:
- you need a domain name, with an 'A' record
- you need administrative access to your server, and concretely:
  - you need to be able to serve files on ports 80 and 443
  - this is because Let's Encrypt uses control over ports 80 and 443 as evidence that you own the machine (anyone can
serve files over ports > 1024)
  - lots of discussion on this topic at https://github.com/letsencrypt/acme-spec/issues/33

## Pre-requisites

- have Docker installed, eg https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-getting-started
- enough bandwidth to install Docker, download the Docker `ubuntu:14.04` image, run `apt-get install git`, and so on.  If you've already
installed docker, and the `ubuntu:14.04` image, about another ~200MB should be sufficient I guess?
- you should have a dns A record pointing at your webserver (eg I use dreamhost for this)
- port 443 should be available on the web server, accessible from the internet
- nothing should be running on port 80 or 443 currently
- so, stop apache if it is running (at least for the duration of this procedure)

## Concepts

- since Let's Encrypt installs a bunch of stuff, rather than hacking around with trying to control this, I simply do
everything from Docker.  Then I dont have to think about what things it's changing on my system.

## Procedure

```
sudo docker run -it ubuntu:14.04 bash
apt-get install -y git bash-completion dnsutils nginx telnet lsof
git clone https://github.com/letsencrypt/letsencrypt
cd letsencrypt
./letsencrypt-auto --help
```

```
# (have installed Docker)
git clone https://github.com/hughperkins/howto-jenkins-ssl
cd howto-jenkins-ssl/docker
sudo docker build -t letsencrypt .
sudo service apache2 stop  # if you are running apache
sudo mkdir -p /etc/letsencrypt
sudo docker run -v /etc/letsencrypt:/etc/letsencrypt -p 80:80 -p 443:443 -it letsencrypt
cd letsencrypt
./letsencrypt-auto certonly --standalone -d myserver.mydomain.com
# (fill in email address)
# (press return twice)
exit
sudo service apache2 start  # if you stopped it earlier
```

