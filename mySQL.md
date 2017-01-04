## 원격 설정하기

1. first check status:
    netstat -tulpen
2. modify your configuration :
    vi /etc/mysql/my.cnf
    bind-address = 0.0.0.0
3. entry mysql to give privileges
    GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'%'IDENTIFIED BY 'mypassword' WITH GRANT OPTION;
    FLUSH PRIVILEGES;
4. restart mysql
    /etc/init.d/mysql restart
    
## edx 설정 

https://github.com/openedx-vlead/port-labs-to-openedx/issues/37    
