# sidecar pattern

centos8

### [1] 먼저 도커 허브에 올릴 이미지를 생성한다.

Dockerfile 작성
```
# Dockerfile
FROM ubuntu:latest
RUN apt update && apt install -y git
ADD ./contents-pull.sh /contents-pull.sh
RUN chmod 755 /contents-pull.sh
WORKDIR /
CMD /contents-pull.sh
```

contents-pull.sh 파일을 작성
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

이미지 빌드
```
docker build --tag [도커허브명]/[레포지토리_이름]:[tag]
```

이미지 도커허브 레포지토리에 업로드
```
docker push [도커허브명]/[레포지토리_이름]:[tag]
```

### [2] Pods 작성
```
apiVersion: v1
kind: Pod
metadata:
  name: sidecar
spec:
  containers:      
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: contents-vol
      
  - name: puller
    image: ehkehk/sidecar:1.0
    env:
    - name: CONTENTS_SOURCE_URL
      value: "https://github.com/MintBANG/side.git"
    volumeMounts:
    - mountPath: /data
      name: contents-vol
      
  volumes:           
  - name: contents-vol
    emptyDir: {}
 ```
 
> "contents-vol"라는 볼륨 안에 nginx, puller 컨테이너
> 
> nginx 컨테이너 : /usr/share/nignx/html 디렉토리를 emptydir로 마운트 -> 컨텐츠를 사용자들이 요청하면 전송
> 
> puller 컨테이너 : git에서 코드를 받아오는 sh를 실행하는 컨테이너
/data 디렉토리를 emptydir로 마운트 -> github에서 1분에 한번씩 컨텐츠 pull


### [3] 사이드 카 IP로 접속하기
```
curl http://[사이드카_IP]
```



