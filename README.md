# E1-1 개발 워크스테이션 구축

## 📋 미션 개요
개발 워크스테이션 환경(Linux CLI · Docker · Git/GitHub)을 직접 구축하고,
모든 과정을 CLI로 수행하며 명령어와 개념을 익히는 것을 목표로 한다.
최종 산출물은 전 과정을 문서화한 본 저장소(README.md)이다.

## 🖥️ 환경 정보
| 항목 | 내용 |
|------|------|
| OS (호스트) | Windows 11 Home |
| 리눅스 | WSL2 · Ubuntu 26.04.1 LTS |
| 커널 | 6.18.54.1-microsoft-standard-WSL2 |
| Docker | 29.9.2 (build a82e528) |
| Git | 2.53.0 |
| 작업 디렉터리 | /home/cari6/codyssey |

## ✅ 체크리스트
- [x] 리눅스 CLI 기본 (pwd, ls, cd, mkdir, touch, cat, echo, cp, mv, rm)
- [x] 파일 권한 이해 및 변경 (chmod 644 / 755 / 700)
- [x] Docker 설치 및 데몬 상태 점검 (docker --version / docker info)
- [x] Docker 이미지 · 컨테이너 조회 (images, ps, ps -a, logs, stats)
- [x] 컨테이너 실행 및 내부 접속 (docker run -it ubuntu bash)
- [x] Dockerfile 직접 작성 및 이미지 빌드 (my-web-server:1.0)
- [x] 포트 매핑 및 접속 검증 (-p 8000:8000 → curl)
- [x] 바인드 마운트 (-v host:container)
- [x] Docker 볼륨 영속성 검증 (컨테이너 삭제 후에도 데이터 유지)
- [x] Git 설정 및 GitHub 연동 (init, add, commit, remote, push)

## 🔍 검증 방법 및 증거

### 1. 리눅스 파일 권한
```bash
$ chmod 644 hello.txt && ls -l hello.txt
-rw-r--r-- 1 cari6 cari6 ... hello.txt
```
권한 숫자(r=4, w=2, x=1)에 따라 10자리 표기가 바뀌는 것을 확인.

### 2. Docker 점검
```bash
$ docker --version
Docker version 29.9.2, build a82e528
```
`docker info`의 Server 섹션이 출력되어 데몬이 정상 동작함을 확인.

### 3. Dockerfile 작성 및 빌드
```dockerfile
FROM ubuntu:latest
RUN apt-get update && apt-get install -y python3
CMD ["python3", "-m", "http.server", "8000"]
```
```bash
$ docker build -t my-web-server:1.0 .
=> naming to docker.io/library/my-web-server:1.0   # 빌드 성공
```

### 4. 포트 매핑 접속 검증
```bash
$ docker run -d -p 8000:8000 --name my-web my-web-server:1.0
$ docker ps
PORTS: 0.0.0.0:8000->8000/tcp
$ curl http://localhost:8000
<title>Directory listing for /</title>   # 웹서버 응답 확인
```

### 5. 바인드 마운트
```bash
$ docker run -d -p 8000:8000 --name my-web -v ~/codyssey/webfiles:/app my-web-server:1.0
$ echo "<h1>New File!</h1>" > ~/codyssey/webfiles/new.html
$ docker exec my-web ls /app
index.html
new.html   # 컨테이너 재시작 없이 호스트 변경이 즉시 반영됨
```

### 6. 볼륨 영속성
```bash
$ docker volume create my-volume
$ docker run -it --name vol-test -v my-volume:/data ubuntu bash
  root@...:/# echo "볼륨 데이터 테스트!" > /data/test.txt
  root@...:/# exit
$ docker rm vol-test
$ docker run --rm -v my-volume:/data ubuntu cat /data/test.txt
볼륨 데이터 테스트!   # 컨테이너를 삭제했음에도 데이터가 유지됨
```

## 🔧 트러블슈팅

### 1) git push 시 `invalid credentials` 인증 실패
- **증상:** push 할 때 GitHub 비밀번호를 입력했으나 `remote: invalid credentials` 오류 발생.
- **원인:** GitHub는 2021년부터 비밀번호 인증을 폐지했으며, Personal Access Token(PAT)을 사용해야 함.
- **해결:** PAT(classic, repo 권한)를 발급한 뒤, `git config --global credential.helper store`로 자격증명을 저장하고 비밀번호 자리에 토큰을 입력하여 push 성공.
- **보안 주의:** 토큰은 저장소·문서·로그 어디에도 노출하지 않으며, 노출 시 즉시 폐기 후 재발급한다.

### 2) PowerShell에서 `docker` 명령을 인식하지 못함
- **증상:** Windows PowerShell에서 `docker` 입력 시 명령을 찾을 수 없음.
- **원인:** Docker가 Windows가 아닌 WSL(Ubuntu) 내부에 설치되어 있어 Windows PATH에 없음.
- **해결:** WSL Ubuntu 터미널 안에서 docker 명령을 실행하여 정상 동작 확인.

## 🔒 보안 유의사항
- 토큰·비밀번호·개인키·인증코드 등 민감정보는 모든 로그·스크린샷·문서에서 마스킹한다.
- 자격증명이 노출된 경우 즉시 삭제하고 재발급한다.
