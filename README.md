# sidecar pattern

centos8

[1] 먼저 도커 허브에 올릴 이미지를 생성한다.


```
# Dockerfile
FROM ubuntu:latest
RUN apt update && apt install -y git
ADD ./contents-pull.sh /contents-pull.sh
RUN chmod 755 /contents-pull.sh
WORKDIR /
CMD /contents-pull.sh
```

contents-pull.sh 파일을 작성한
```
#!/bin/bash

if [ -z $CONTENTS_SOURCE_URL ]; then
   exit 1
fi

git clone $CONTENTS_SOURCE_URL /data

cd /data
while true
do
   date
   sleep 60
   git pull
done
```
