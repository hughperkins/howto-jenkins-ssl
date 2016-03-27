# howto-jenkins-ssl
quick how to on activating ssl on jenkins, so I can find it easily

New!  Alternative procedure, using Lets Encrypt certificate, available now.  See [letsencrypt.md](letsencrypt.md).

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

# start jenkins

* if you want both https and http:

```
java -jar jenkins.war --httpsPort=8443 --httpsCertificate=cert.pem --httpsPrivateKey=key.pem
```

* if you want https only, dont open http port:

```
java -jar jenkins.war --httpsPort=8443 --httpsCertificate=cert.pem --httpsPrivateKey=key.pem --httpPort=-1
```

# starting a slave

* Convert the cert.pem to cert.der:
```
 openssl x509 -outform der -in cert.pem -out cert.der
```

* create keystore, containing this cert:

```
keytool -import -alias testweb.local -keystore cacerts -file cert.der
# reply trust certificate=yes
# put keystore password of 'changeit', or make your own password
```
* transfer this file to the slave computer somehow (eg via /var/www/html, and download from slave)
* launch slave
  * as for normal slave launch, but add `-Djavax.net.trustStore=cacerts
```
java -Djavax.net.ssl.trustStore=cacerts -jar slave.jar -jnlpUrl https://jenkins.myweb.com:8443/computer/testnode/slave-agent.jnlp
```
=> will work ok :-)
