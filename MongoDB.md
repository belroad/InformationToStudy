## mongoDB 설치하기
  
  1. mongoDB 설치파일 다운로드 <br/>
     curl -O https://fastdl.mongodb.org/osx/mongodb-osx-x86_64-3.4.1.tgz <br/>
  2. mongoDB 설치파일 압축해제 <br/>
     tar -zxvf mongodb-osx-x86_64-3.4.1.tgz <br/>
  3. mongoDB 압축 해제한 폴더를 이동 <br/>
     mkdir -p mongodb ( 옵션 -p: 중간계층의 폴더 생성가능하게 한다. ex) a / b / c ) <br/>
     cp -R -n mongodb-osx-x86_64-3.4.1/ mongodb ( 옵션 -R: 복사하려는 폴더의 하위까지 복사한다. ) <br/>
  4. mongoDB의 bin폴더를 환경변수로 등록 (osx의 환경변수 등록법 참고사이트: http://osxtip.tistory.com/632 ) <br/>
     export PATH=<mongodb-install-directory>/bin:$PATH <br/>
  5. mongoDB의 데이터를 저장하기 위한 폴더 설정 (기본위치: /data/db ) <br/>
     mkdir -p /data/db <br/>
  6. mongoDB의 데이터를 저장하기 위한 폴더 권한 설정 (참고사이트: http://mintnlatte.tistory.com/258 ) <br/>
     chmod 777 data <br/>
  7. mongoDB 실행 <br/>
     1) /mongodb/bin/mongod  존재하고 데이터 저장 폴더가 /data/db 일 경우 <br/>
         mongod <br/>
     2) /mongodb/bin/mongod  아닌 다른 폴더일 경우 데이터 저장 폴더가 /data/db 일 경우 <br/>
         <path to binary>/mongod <br/>
     3) /mongodb/bin/mongod  존재하고 데이터 저장 폴더가 /data/db 가 아닐 경우 <br/>
         mongod —dbpath <path to data directory> <br/>
     


https://docs.mongodb.com/v3.4/core/databases-and-collections/

https://velopert.com/category/dev-log/tech-log/mongodb
