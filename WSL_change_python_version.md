# Python version 변경
* WSL에 설치된 default python 을 Upgrade 하고 싶다.
---
1. Python 설치

2. Python version 변경

``` bash
# 'which python' 미수행되는 경우 있음
bwyoon@LAPTOP-BBDBCK09:/usr/bin$ ls -al python3
lrwxrwxrwx 1 root root 10 Aug 18 19:39 python3 -> python3.10
bwyoon@LAPTOP-BBDBCK09:/usr/bin$ ls -al python3.10
-rwxr-xr-x 1 root root 5921160 Nov 15 01:10 python3.10
bwyoon@LAPTOP-BBDBCK09:/usr/bin$ ls -al python3.11
-rwxr-xr-x 1 root root 6890080 Aug 12 19:02 python3.11
# Symbolic link 변경
bwyoon@LAPTOP-BBDBCK09:/usr/bin$ ln -Tfs python3.11 python3
ln: failed to create symbolic link 'python3': Permission denied
bwyoon@LAPTOP-BBDBCK09:/usr/bin$ sudo ln -Tfs python3.11 python3
[sudo] password for bwyoon:
bwyoon@LAPTOP-BBDBCK09:/usr/bin$ ls -al python3
lrwxrwxrwx 1 root root 10 Jan  9 02:05 python3 -> python3.11
```
---
## Reference
* [Python 심볼릭 링크 설정](https://eehoeskrap.tistory.com/316)
* [우분투에서 파이썬 버전 변경하기](https://seongkyun.github.io/others/2019/05/09/ubuntu_python/)