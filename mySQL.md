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



CURRENT_VERSION = '1'
SEPARATOR = u"|"

cls._validate_cookie_params(session_id, user_id)

 @staticmethod
 def _validate_cookie_params(session_id, user_id):
    """
    Validates the given parameters for cookie creation.

    Raises SafeCookieError if session_id is None.
    """
    # Compare against unicode(None) as well since the 'value'
    # property of a cookie automatically serializes None to a
    # string.
    if not session_id or session_id == unicode(None):
        # The session ID should always be valid in the cookie.
        raise SafeCookieError(
            "SafeCookieData not created due to invalid value for session_id '{}' for user_id '{}'.".format(
                session_id,
                user_id,
            ))

    if not user_id:
        # The user ID is sometimes not set for
        # 3rd party Auth and external Auth transactions
        # as some of the session requests are made as
        # Anonymous users.
        log.debug(
            "SafeCookieData received empty user_id '%s' for session_id '%s'.",
            user_id,
            session_id,
        )

    safe_cookie_data = SafeCookieData(
        cls.CURRENT_VERSION,
        session_id,
        key_salt=get_random_string(),
        signature=None,
    )
    safe_cookie_data.sign(user_id)
    return safe_cookie_data
    
    
    def get_random_string(length=12,
                      allowed_chars='abcdefghijklmnopqrstuvwxyz'
                                    'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789'):
    """
    Returns a securely generated random string.

    The default length of 12 with the a-z, A-Z, 0-9 character set returns
    a 71-bit value. log_2((26+26+10)^12) =~ 71 bits
    """
    if not using_sysrandom:
        # This is ugly, and a hack, but it makes things better than
        # the alternative of predictability. This re-seeds the PRNG
        # using a value that is hard for an attacker to predict, every
        # time a random string is required. This may change the
        # properties of the chosen random sequence slightly, but this
        # is better than absolute predictability.
        random.seed(
            hashlib.sha256(
                ("%s%s%s" % (
                    random.getstate(),
                    time.time(),
                    settings.SECRET_KEY)).encode('utf-8')
            ).digest())
    return ''.join(random.choice(allowed_chars) for i in range(length))
    
    allowed_chars = u'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789'
    
    def choice(self, seq):
        """Choose a random element from a non-empty sequence."""
        return seq[int(self.random() * len(seq))]  # raises IndexError if seq is empty
    
     def random(self):
        """Get the next random number in the range [0.0, 1.0)."""
        return (long(_hexlify(_urandom(7)), 16) >> 3) * RECIP_BPF
        
         def sign(self, user_id):
        """
        Computes the signature of this safe cookie data.
        A signed value of hash(version | session_id | user_id):timestamp
        with a usage-specific key derived from key_salt.
        """
        data_to_sign = self._compute_digest(user_id)
        self.signature = signing.dumps(data_to_sign, salt=self.key_salt)        
        
        
    def _compute_digest(self, user_id):
        """
        Returns hash(version | session_id | user_id |)
        """
        hash_func = sha256()
        for data_item in [self.version, self.session_id, user_id]:
            hash_func.update(unicode(data_item))
            hash_func.update('|')
        return hash_func.hexdigest() 
        
        
        def _compute_digest(self, user_id):
        """
        Returns hash(version | session_id | user_id |)
        """
        hash_func = sha256()
        for data_item in [self.version, self.session_id, user_id]:
            hash_func.update(unicode(data_item))
            hash_func.update('|')
        return hash_func.hexdigest()
        
            def _compute_digest(self, user_id):
        """
        Returns hash(version | session_id | user_id |)
        """
        hash_func = sha256()
        for data_item in [self.version, self.session_id, user_id]:
            hash_func.update(unicode(data_item))
            hash_func.update('|')
        return hash_func.hexdigest()
        
        
        
        
