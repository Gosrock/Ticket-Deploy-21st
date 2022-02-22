# Ticket-Deploy

# 프로젝트 구조

![서버세팅](https://user-images.githubusercontent.com/13329304/155071160-396f3344-1627-4df7-980e-5624b3804468.jpg)

각 레포지토리에서 github action을 통해 도커이미지를 빌드합니다.

특히 react 를 사용하는 레포지토리는

```yaml
FROM node:14 AS builder
WORKDIR /app

COPY package-lock.json ./
COPY package.json ./
RUN npm ci

COPY . ./
RUN npm run build

FROM nginx:1.19.0
COPY nginx/default.conf /etc/nginx/conf.d/default.conf
RUN rm -rf ./usr/share/nginx/html/*
COPY --from=builder /app/build /usr/share/nginx/html/

ENTRYPOINT ["nginx", "-g", "daemon off;"]
```

react build 된 결과물을 nginx를 웹서버로 활용해서 도커이미지로 빌드합니다.

## 배포
main branch에 push 액션이 들어오게 되면 github action을 동작시킵니다.
초기 [deploy.sh](https://github.com/Gosrock/Ticket-Deploy/blob/main/deploy.sh) 파일에서 도커가 깔려있는지 확인과 설치를 진행하고, ssh 커맨드를 통해 docker-compose를 실행시킵니다.

## 환경변수
환경변수는 .env 파일 형태로 전달합니다.
github setting 에서 env를 설정한뒤에 해당 env들을 파일형태로 action이 실행되었을때 넘겨줍니다.

## 로깅
로깅은 aws cloudwatch 와 awslogger을 사용합니다.
[관련 포스팅](https://devnm.tistory.com/8)
