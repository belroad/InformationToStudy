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

## edx 세션 생성

safe_session/middleware.py

edx 기본 언어코드가 비로그인시에도 en으로 들어가 있음.

key_salt = "django.contrib.auth.models.AbstractBaseUser.get_session_auth_hash"
password = 

==> 0cf2e4a1d56e02e5accbde74ce9d5c59f354fdb3

{u'_language': u'en'}

{u'_language': u'en', '_auth_user_hash': '0cf2e4a1d56e02e5accbde74ce9d5c59f354fdb3', '_auth_user_backend': 'ratelimitbackend.backends.RateLimitModelBackend', '_auth_user_id': u'5'}
