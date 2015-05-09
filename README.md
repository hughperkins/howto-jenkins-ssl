# howto-jenkins-ssl
quick how to on activating ssl on jenkins, so I can find it easily

# given:

- your website is at jenkins.myweb.com
- have openssl installed

# generate key

```
openssl genrsa -out key.pem  # creates key.pem

openssl req -new -key key.pem -out csr.pem
# you need to put the dns name of your website, testweb.local
# for the 'Common Name' question
# other questions, you can just accept defaults
# actually, you can accept defaults for all, will work ok too

openssl x509 -req -days 9999 -in csr.pem -signkey key.pem -out cert.pem
rm csr.pem
```

# start jenkins, with https, and http still available

```
java -jar jenkins.war --httpsPort=8443 --httpsCert=cert.pem --httpsKey=key.pem
```

# start jenkins, with https, http not available

```
java -jar jenkins.war --httpsPort=8443 --httpsCert=cert.pem --httpsKey=key.pem --httpPort=-1
```
