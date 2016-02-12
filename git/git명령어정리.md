# git 정리

### 로컬의 소스를 원격소스로 강제로 체크아웃하기 (svn revert)

```
$ git checkout -f
```

혹은

```
$ git reset --hard HEAD
```

위에꺼가 좀 더 이해하기 편한듯

###  git로그 한줄로 보기

```
$ git log --pretty=online
```

### git로그 한줄로 보기 + 7자리 해쉬로 보기

```
$ git log --oneline
```
혹은
```
$ git log --pretty=oneline --abbrev-commit
```

### git로그 짧게보기 설정하기

```
$ git config --global alias.slog "log --oneline"
```

위와 같이 설정하면

```
$ git slog
```
로 짧게 볼 수 있다.

$ 커밋해쉬 짧은거 얻어오기

```
$ git rev-parse --short HEAD
```

### 최신 커밋의 로그 보기

```
$ git branch -v
```
