# Project 만들고 Git 설정하는 프로세스
1. Project 폴더 생성
* GitHub repository 생성
* Local (Winodw/WSL) 에 동일한 이름으로 생성한다.
2. .gitignore 생성 (option)
* ex) .venv 와 같은 파이썬 가상환경을 구성하고 ignore 추가하려면 미리 .gitignore 생성되어 있어야 함

``` bash
bwyoon@LAPTOP-BBDBCK09:~$ ls
Workspace
# Project 폴더 생성
bwyoon@LAPTOP-BBDBCK09:~$ mkdir Document
bwyoon@LAPTOP-BBDBCK09:~$ cd Document/
bwyoon@LAPTOP-BBDBCK09:~/Document$ ls
# git init
bwyoon@LAPTOP-BBDBCK09:~/Document$ git init
Initialized empty Git repository in /home/bwyoon/Document/.git/
# create .gitignore
# .venv 와 같은 파이썬 가상환경을 구성하고 ignore 추가하려면 미리 .gitignore 생성되어 있어야 함
bwyoon@LAPTOP-BBDBCK09:~/Document$ touch .gitignore
# vscode 실행
bwyoon@LAPTOP-BBDBCK09:~/Document$ code .
bwyoon@LAPTOP-BBDBCK09:~/Document$ git remote -v
# git remote 설정
bwyoon@LAPTOP-BBDBCK09:~/Document$ git branch -M main
bwyoon@LAPTOP-BBDBCK09:~/Document$ git remote add origin https://github.com/ByoungwooYoon/Document.git
bwyoon@LAPTOP-BBDBCK09:~/Document$ git remote -v
origin  https://github.com/ByoungwooYoon/Document.git (fetch)
origin  https://github.com/ByoungwooYoon/Document.git (push)
bwyoon@LAPTOP-BBDBCK09:~/Document$
```