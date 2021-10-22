# Network
네트워크 및 보안 공부

## Git Push

로컬 저장소 생성
```
$ git init
```

현재 디렉토리에 있는 파일 확인 
```
$ git status
```

현재 로컬 저장소에 있는 파일 전부 올리기 
```
$ git add .
```

커밋 [push 메시지명]
```
$ git commit -m "push message"
```

기존 워킹 디렉토리에 새 리모트 저장소 추가 
```
git remote add origin [repository address]
```

로컬 저장소를 원격 저장소에 연결
```
$ git remote -v
```
푸쉬
```
$ git push origin master
```
