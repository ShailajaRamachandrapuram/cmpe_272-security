# cmpe_272-security

Homework #7

Design and build a PKI infrastructure

Student name: Shailaja Ramachandrapuram

University: San Jose State University

Step 1: Create a one new directory.

git clone https://bitbucket.org/stefanholek/pki-example-1

cd pki-example-1

<img width="468" alt="image" src="https://user-images.githubusercontent.com/111623287/201494213-d093ff19-abc9-4d0e-adea-f3f79229d3c1.png">

Step 2: Create Root CA directory holds CA resources, CRLs directory and certs directory holds user certificates. And create database.

mkdir -p ca/root-ca/private ca/root-ca/db crl certs
chmod 700 ca/root-ca/private

cp /dev/null ca/root-ca/db/root-ca.db

cp /dev/null ca/root-ca/db/root-ca.db.attr

echo 01 > ca/root-ca/db/root-ca.crt.srl

echo 01 > ca/root-ca/db/root-ca.crl.srl

Create CA request.

openssl req -new \

    -config etc/root-ca.conf \
    
    -out ca/root-ca.csr \
    
    -keyout ca/root-ca/private/root-ca.key
    
Create CA certificate.

openssl ca -selfsign \
    -config etc/root-ca.conf \
    -in ca/root-ca.csr \
    -out ca/root-ca.crt \
    -extensions root_ca_ext
    
<img width="468" alt="image" src="https://user-images.githubusercontent.com/111623287/201494279-1f02c495-fa37-4bcc-8dbe-eb1dd72083a9.png">

Step 3: Create signing CA ca directory holds CA resources, CRLs, and certificates.

mkdir -p ca/signing-ca/private ca/signing-ca/db crl certs

chmod 700 ca/signing-ca/private

Create signing database.

cp /dev/null ca/signing-ca/db/signing-ca.db

cp /dev/null ca/signing-ca/db/signing-ca.db.attr

echo 01 > ca/signing-ca/db/signing-ca.crt.srl

echo 01 > ca/signing-ca/db/signing-ca.crl.srl

Create signing CA request.

openssl req -new \

    -config etc/signing-ca.conf \
    
    -out ca/signing-ca.csr \
    
    -keyout ca/signing-ca/private/signing-ca.key

<img width="493" alt="image" src="https://user-images.githubusercontent.com/111623287/201494344-6138842a-52e8-4eed-8d69-b9834db4801f.png">

<img width="493" alt="image" src="https://user-images.githubusercontent.com/111623287/201494348-cd7c86f3-3ba1-4b5d-a78e-bfe632d0b838.png">
 
Create signing CA certificate.

openssl ca \

    -config etc/root-ca.conf \
    
    -in ca/signing-ca.csr \
    
    -out ca/signing-ca.crt \
    
    -extensions signing_ca_ext

<img width="468" alt="image" src="https://user-images.githubusercontent.com/111623287/201494361-3dfe04d9-5568-46f2-8da5-90ea0959d312.png">

<img width="468" alt="image" src="https://user-images.githubusercontent.com/111623287/201494370-1abf7c4e-9e94-4e68-8eb5-720570dbfb06.png">

<img width="468" alt="image" src="https://user-images.githubusercontent.com/111623287/201494374-df628218-9f06-4a7c-b919-8ba0b8636444.png">

Step 4: Create TLS server request

SAN=DNS:www.simple.org \

openssl req -new \

    -config etc/server.conf \
    
    -out certs/simple.org.csr \
    
    -keyout certs/simple.org.key

<img width="468" alt="image" src="https://user-images.githubusercontent.com/111623287/201494381-7dc2b5cc-d71f-4ac9-91ae-77379cfc8c35.png">

Create TLS server Certificate.

openssl ca \

    -config etc/signing-ca.conf \
    
    -in certs/simple.org.csr \
    
    -out certs/simple.org.crt \
    
    -extensions server_ext
<img width="468" alt="image" src="https://user-images.githubusercontent.com/111623287/201494402-7457183e-fea1-476e-b60c-55b9c6b8047b.png">

Step 5: Install tomcat on TLS server certificate.

Prepare keystore file and add root and signing certificate.

keytool -import -alias root-ca -keystore certs/simple.jks -trustcacerts -file ca/root-ca.crt

keytool -import -alias signing-ca -keystore certs/simple.jks -trustcacerts -file ca/signing-ca.crt

Step 6: Generate certificate-key pair for server.

openssl pkcs12 -export -name "tomcat" -inkey certs/simple.org.key -in certs/simple.org.crt -out certs/simple.p12

Once keystore is ready, then in Tomcat, SSL configuration needs to be made in “server.xml” which is in the “conf” directory of the Tomcat installation.

<Connector protocol="org.apache.coyote.http11.Http11NioProtocol"
    
    sslImplementationName="org.apache.tomcat.util.net.jsse.JSSEImplementation"
    
    port="8443" maxThreads="150" SSLEnabled="true" scheme="https"
    
    secure="true" keyAlias="tomcat"  keystoreFile="/home/ubuntu/pki-example-1/certs/simple.jks"
    
    keystorePass="changeit" clientAuth="false" sslProtocol="TLS" />

<img width="468" alt="image" src="https://user-images.githubusercontent.com/111623287/201494439-62184bb7-39e9-4c36-92b8-365953c9dbb6.png">

<img width="468" alt="image" src="https://user-images.githubusercontent.com/111623287/201494445-73532aa1-8215-4f31-bec6-c78f98c4fe4b.png">


 
