# Basic Command 

```shell
docker build -t dingodb.deploy.ubuntu:cloud .
docker images
docker tag 295ed13db2eb 172.20.3.185:5000/dingodb/dingodb.ubuntu:cloud
docker push 172.20.3.185:5000/dingodb/dingodb.ubuntu:cloud
```
