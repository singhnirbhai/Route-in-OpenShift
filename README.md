
# Route in OpenShift
## create certificates with route hostname
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key \
  -out tls.crt \
  -subj "/CN=httpd-ssl-https.apps-crc.testing/O=example"
```
## create secret with certificates 
```bash
oc create secret tls https-certs --cert tls.crt --key tls.key
```
## create ssl.conf file
```bash
vi ssl.conf
```
```bash
# Include this file in Apache config
IncludeOptional /etc/httpd-extra/*.conf

LoadModule ssl_module modules/mod_ssl.so
Listen 8443

<VirtualHost *:8443>
    DocumentRoot "/var/www/html"
    ServerName example.com

    SSLEngine on
    SSLCertificateFile "/etc/httpd/ssl/tls.crt"
    SSLCertificateKeyFile "/etc/httpd/ssl/tls.key"

    <Directory "/var/www/html">
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

## create deployment 
```bash
vi https.yml
```
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-ssl
spec:
  replicas: 1
  selector:
    matchLabels:
      app: httpd-ssl
  template:
    metadata:
      labels:
        app: httpd-ssl
    spec:
      containers:
      - name: httpd
        image: registry.redhat.io/rhel8/httpd-24:latest
        ports:
        - containerPort: 8443
        volumeMounts:
        - name: ssl-cert
          mountPath: "/etc/httpd/ssl"
          readOnly: true
        - name: ssl-conf
          mountPath: /etc/httpd/conf.d/custom-ssl.conf
          subPath: ssl.conf
      volumes:
      - name: ssl-cert
        secret:
          secretName: todo-certs
      - name: ssl-conf
        configMap:
          name: httpd-ssl-conf
 ```
 ## create service
## create servcie file
 ```bash
 vi service.yml
 ```
```bash
apiVersion: v1
kind: Service
metadata:
  name: httpd-ssl-svc
spec:
  selector:
    app: httpd-ssl
  ports:
  - port: 8443
    targetPort: 8443          
```
## check deployment
```bash
oc get all
```
## Create passthrough route 
```bash
oc create route passthrough httpd-ssl --service=httpd-ssl-svc 
```
```bash
oc get route 
```
## For troubleshooting certificate inside the container
```bash
openssl s_client -connect httpd-ssl-https.apps-crc.testing:8443 -showcerts
```
## check certificate inside the container
```bash
cd /etc/httpd/ssl/
```
## check certicate inside the container
```bash
openssl s_client -connect localhost:8443 -showcerts
```








