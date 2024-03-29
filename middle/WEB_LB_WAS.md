# WEB_LB_WAS 연동

## Apache
setsebool -P httpd_can_network_connect on

### httpd.conf
```
LogFormat "%{X-Forwarded-For}i %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined

Include conf/extra/httpd-vhosts.conf
Include conf/extra/mod_remoteip.conf
```

### httpd-vhosts.conf
```
<VirtualHost *:8080>
   DocumentRoot "/var/www/html2"
   ServerName test.com
   ProxyPass / http://${LB_IP_PORT}/ disablereuse=on
   ProxyPassReverse / http://${LB_IP_PORT}/
</VirtualHost>
```

### mod_remoteip.conf
```
LoadModule remoteip_module modules/mod_remoteip.so
RemoteIPHeader X-Forwarded-For
```

## Tomcat

### server.xml
```
....
    <!-- You should set jvmRoute to support load-balancing via AJP ie :
    <Engine name="Catalina" defaultHost="localhost" jvmRoute="jvm1">
    -->
    <Engine name="Catalina" defaultHost="localhost">


        <Valve className="org.apache.catalina.valves.RemoteIpValve"
             internalProxies="\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}"
             remoteIpHeader="x-forwarded-for"
             proxiesHeader="x-forwarded-by"
             protocolHeader="x-forwarded-proto" />
      <!--For clustering, please take a look at documentation at:
          /docs/cluster-howto.html  (simple how to)
          /docs/config/cluster.html (reference documentation) -->
....
        <!-- Access log processes all example.
             Documentation at: /docs/config/valve.html
             Note: The pattern used is equivalent to using pattern="common" -->
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%{X-Forwarded-For}i %h %l %u %t &quot;%r&quot; %s %b" />

      </Host>
    </Engine>
  </Service>
</Server>
```
