# struts_poc

git clone https://github.com/hook-s3c/apache-struts2-PoC

cd apache-struts2-PoC/

docker build -t struts .

docker run -it --rm -d -p 8080:8080 struts:latest

docker tag struts:latest public.ecr.aws/v9f6i2g3/stuff:latest

#public login to us-west-2 not supported by aws

aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws

docker push public.ecr.aws/v9f6i2g3/stuff:latest

curl 127.0.0.1:8080/showcase.action -v

#python3 -m venv .

python exploitS2-048-cmd.py 127.0.0.1:8080 'cat /etc/passwd'
