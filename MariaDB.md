## ubuntu12.04 mariaDB 설치하기

(ubuntu12.04)
 
sudo apt-get install python-software-properties
 
sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xcbcb082a1bb943db
 
sudo add-apt-repository 'deb [arch=amd64,i386] http://ftp.kaist.ac.kr/mariadb/repo/5.5/ubuntu precise main'
 
sudo apt-get update
 
sudo apt-get install mariadb-server

## 마리아DB 테이블 및 계정생성
 
mysql -u root -p
 
database 확인 : show databases;
 
database 생성: create database [database name];
 
database 사용: use [database name];
 
database 삭제: drop [database name];
 
database내 table 확인: show tables;
 
사용자 계정 확인: select host, user from user;
 
사용자 계정 생성:
    grant usage on [database명].[table명] to [user명]@[server명] identified by [password]; <br/>
    ex) grant usage on database.* to belroad@localhost identified by belroad;
 
생성된 사용자 계정 권한 설정
    GRANT ALL ON [database명].[table명] TO [user명]@[server명];    =>  모든 권한을 준다
    GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,INDEX,ALTER ON [database명].[tabel명] TO [user명]@[server명];   => 특정 권한을 준다
    ex) grant all on database.* to user1@localhost;
        grant select,insert,update,delete,create,drop.index,alter on database.* to user1@localhost;
 
사용자 계정 권한 삭제
    REVOKE ALL ON [database명].[table명] FROM [user명]@[server명];    =>  모든 권한을 삭제한다
    REVOKE DROP ON [database명].[table명] FROM [user명]@[server명];    => 특정 권한(drop)을 삭제한다
    ex) revoke all on database.* from user1@localhost;
        revoke drop,index on database.* from user1@localhost;
 
계정 권한을 새로 로드
FLUSH PRIVILEGES;
 
사용자 계정 삭제
DROP USER [user명]@[server명];
ex) drop user user1@localhost;

## 멤캐시 설치
1. 멤캐시 설치
    1) sudo apt-get install memcached
    2) sudo vi /etc/memcached.conf
        -l 0.0.0.0
    3) sudo service memcached restart
 
2. 멤캐시 설치완료 테스트
    telnet <ipaddress> 11211
 
3. memcached binding 을 위해 python-memcached 설치
    pip install python-memcached
 
4. settings.py 에 등록
 
    CACHES = {
	    ‘default’: {
		‘BACKEND’: ‘django.core.cache.backends.memcached.MemcachedCache’,
		‘LOCATION’: ’127.0.0.1:11211’,
	     }
         }
