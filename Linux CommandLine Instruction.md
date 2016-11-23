# Linux CommandLine Instruction

###### 전체 폴더에서 찾기:
```
  find / -name 폴더명 -type d
```  

###### 현재 폴더(및 하위폴더)에서 찾기:
```
  find ./ -name 폴더명 -type d
```  

###### 단축키 :
```
  ctrl + L  : 현재 입력 글자들은 남겨둔 채 'clear' 실행
  ctrl + U  : 현재 커서 위치부터 그 줄 처음부분 까지 지우기 
```
## 환경변수 설정 
###### 환경변수 설정
```
export 환경변수명=경로
```
###### 환경변수 설정 확인
```
env|grep 환경변수명
```
###### 환경변수 모든사용자 항상 적용
```
/etc/bash.bashrc 에 정의
```
###### 특정사용자 항상 적용
```
/home/userName/.bashrc
```
###### 환경변수 해제
```
unset 환경변수명
```
