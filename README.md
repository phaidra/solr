# Adding new core

1) Create core
```bash
mkdir /var/solr/data/newcore
cp -R /var/solr/data/someoldcore/conf /var/solr/data/newcore/
mkdir /var/solr/data/newcore/data
sudo chown solr:solr -R /var/solr/data/newcore/

```

2) Add newcore in Solr's core manager

# Securing Solr (jetty)

### HashLoginService

#### 6.1

Add 
```xml
    <Call name="addBean">
      <Arg>
        <New class="org.eclipse.jetty.security.HashLoginService">
          <Set name="name">Solr Admin Access</Set>
          <Set name="config"><SystemProperty name="jetty.home" default="."/>/etc/realm.properties</Set>
          <Set name="refreshInterval">0</Set>
        </New>
      </Arg>
   </Call>
```
to jetty.xml (/opt/solr-6.1.0/server/etc/jetty.xml)

#### 7.3

Add 
```xml
    <Call name="addBean">
      <Arg>
        <New class="org.eclipse.jetty.security.HashLoginService">
          <Set name="name">Solr Admin Access</Set>
          <Set name="config"><SystemProperty name="jetty.home" default="."/>/etc/realm.properties</Set>
        </New>
      </Arg>
   </Call>
```
to jetty.xml (/opt/solr-7.3.1/server/etc/jetty.xml)

### jetty users

Generate password hash
```bash
java -cp /opt/solr-6.1.0/server/lib/jetty-util-9.3.8.v20160314.jar org.eclipse.jetty.util.security.Password myusername mysecret

2017-03-27 15:37:34.752:INFO::main: Logging initialized @173ms
mypassword
OBF:1uh41zly1x8g1vu11ym71ym71vv91x8e1zlk1ugm
MD5:34819d7beeabb9260a5c854bc85b3e44
CRYPT:myylAylKPNtmw
```

Create realm.properties (as configured in HashLoginService, eg /etc/realm.properties points to /opt/solr-6.1.0/server/etc/realm.properties). Use the hash (eg OBF) as generated before. The syntax is
username: hash[,role]
```java
myusername: OBF:1uh41zly1x8g1vu11ym71ym71vv91x8e1zlk1ugm,admin
```

### web.xml

Add security-constraint with auth-constraint for everything and a security-constraint without auth-constraint for stuff which should be accessed publicly.

```xml
  <security-constraint>
    <web-resource-collection>
      <web-resource-name>mysolrcore-select</web-resource-name>
      <url-pattern>/mysolrcore/select/*</url-pattern>
    </web-resource-collection>
    <web-resource-collection>
      <web-resource-name>mysolrcore-pages-select</web-resource-name>
      <url-pattern>/mysolrcore_pages/select/*</url-pattern>
    </web-resource-collection>
    <web-resource-collection>
      <web-resource-name>mysolrcore-suggest</web-resource-name>
      <url-pattern>/mysolrcore/suggest/*</url-pattern>
    </web-resource-collection>
    <web-resource-collection>
      <web-resource-name>mysolrcore-export</web-resource-name>
      <url-pattern>/mysolrcore/export/*</url-pattern>
    </web-resource-collection>
  </security-constraint>

  <security-constraint>
    <web-resource-collection>
      <web-resource-name>Solr authenticated application</web-resource-name>
      <url-pattern>/*</url-pattern>
    </web-resource-collection>
    <auth-constraint>
      <role-name>admin</role-name>
    </auth-constraint>
  </security-constraint>

  <login-config>
    <auth-method>BASIC</auth-method>
    <realm-name>Solr Admin Access</realm-name>
  </login-config>
```

### log4shell

zip -d log4j-core-*.jar org/apache/logging/log4j/core/lookup/JndiLookup.class

### restart solr service

# Changing certificate

```bash
openssl pkcs12 -export -in /etc/ssl/certs/myhostname.crt -inkey /etc/ssl/private/myhostname.key -name solr -certfile chain.crt -out myhostname.p12

sudo keytool -importkeystore -deststorepass secret -destkeystore /opt/solr-6.1.0/server/etc/solr-ssl.keystore.jks -srckeystore /etc/ssl/private/myhostname.p12 -srcstoretype PKCS12

service solr restart
```
