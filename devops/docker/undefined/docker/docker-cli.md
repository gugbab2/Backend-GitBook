# 현업에서 자주 사용하는 Docker CLI 익히기

## 이미지 다운로드&#x20;

* 이미지를 다운로드 할 때, Dockerhub 라는 곳에서 이미지를 다운받는다.&#x20;

#### 최신

```bash
# docker pull 이미지명 
docker pull nginx # docker pull nginx:latest 와 동일하게 작동
```

#### 특정 버전 이미지 다운로드

```shellscript
# docker pull 이미지명:태그명
docker pull nginx:stable-perl
```

## 이미지 조회 / 삭제&#x20;

#### 다운받은 모든 이미지 조회&#x20;

```shellscript
docker image ls
```

#### 이미지 삭제&#x20;

<pre class="language-shellscript"><code class="lang-shellscript"><strong># 특정 이미지 삭제
</strong><strong>docker image rm [이미지ID 또는 이미지명]
</strong><strong>
</strong><strong># 중지된 컨테이너에서 사용하고 있는 이미지 강제 삭제하기 
</strong><strong>docker image rm -f [이미지 ID 또는 이미지명]
</strong><strong>
</strong><strong># 전체 이미지 삭제 
</strong><strong>## 컨테이너에서 사용하고 있지 않은 이미지만 전체 삭제
</strong><strong>docker image rm $(docker images -q)
</strong><strong>## 컨테이너에서 사용하고 있는 이미지를 포함해서 전체 이미지 삭제 
</strong><strong>docker image rm -f $(docker images -q)
</strong><strong>
</strong></code></pre>

## 컨테이너 실행1&#x20;

#### 컨테이너 생성&#x20;

```shellscript
# docker create 이미지명[:태그명]
docker create nginx

docker ps -a # 모든 컨테이너 조회 
```

#### 컨테이너 실행&#x20;

```shellscript
# docker start 컨테이너명[또는 컨테이너 ID]
docker start 컨테이너명[또는 컨테이너 ID]

# 실행중인 컨테이너 조회 
docker ps 

# Nginx 컨테이너 중단 후 삭제하기 
docker ps 
docker stop 컨테이너명[또는 컨테이너 ID]
docker rm 컨테이너명[또는 컨테이너 ID]
docker image rm 이미지명[또는 이미지 ID]
```

## 컨테이너 실행2&#x20;

#### 이미지를 바탕으로 컨테이너를 생성한 뒤, 컨테이너를 실행

```shellscript
# docker run 이미지명[:태그명]
docker run nginx # 포그라운드 실행
docker run -d nginx # 백그라운드 실행 

# Nginx 컨테이너 중단 후 삭제하기 
docker ps 
docker stop {nginx 실행시킨 컨테이너 ID}
docker rm {nginx 실행시킨 컨테이너 ID}
docker image rm nginx 
```

#### 컨테이너에 이름 붙여서 생성 및 실행

```shellscript
docker run -d --name my-web-server nginx

# Nginx 컨테이너 중단 후 삭제하기 
docker ps 
docker stop {nginx 실행시킨 컨테이너 ID}
docker rm {nginx 실행시킨 컨테이너 ID}
docker image rm nginx 
```

#### 호스트의 포트와 컨테이너의 포트를 연결하기&#x20;

```shellscript
docker run -d -p 4000:80 nginx
```

<figure><img src="../../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

## 컨테이너 조회 / 중지 / 삭제&#x20;

#### 컨테이너 조회&#x20;

```shellscript
docker ps 
docker ps -a 
```

#### 컨테이너 중지&#x20;

```shellscript
docker stop 컨테이너명[또는 컨테이너 ID] # 정상종료
docker kill 컨테이너명[또는 컨테이너 ID] # 강제종료
```

#### 컨테이너 삭제&#x20;

```shellscript
docker rm 컨테이너명[또는 컨테이너 ID]
docker rm -f 컨테이너명[또는 컨테이너 ID]
docker rm $(docker ps -qa) # 중지되어 있는 모든 컨테이너 삭제 
docker rm of $(docker ps -qa) # 실행되고 있는 모든 컨테이너 삭제
```

## 컨테이너 로그 조회&#x20;

#### 특정 컨테이너 모든 로그 조회

```shellscript
docker run -d nginx 
docker logs [nginx 가 실행되고 있는 컨테이너 ID]
```

#### 최근 로그 10줄만 조회&#x20;

```shellscript
docker logs --tail 10 [컨테이너 ID 또는 컨테이너명]
```

#### 기존 로그 조회 + 생성되는 로그를 실시간으로 보고 싶은 경우

```shellscript
docker run -d -p 80:80 nginx 
docker logs -f
```

#### 기존 로그는 조회하지 않기 + 생성되는 로그를 실시간으로 보고 싶은 경우&#x20;

```shellscript
docker logs --tail 0 -f [컨테이너 ID 또는 컨테이너명]
```

## 실행중인 컨테이너 내부에 접속하기(exec -it)

```shellscript
docker run -d nginx 
docker exec -it [Nginx 가 실행되고 있는 컨테이너 ID] bash 
exit
```
