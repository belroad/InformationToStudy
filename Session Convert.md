## 멤캐시 저장되는 아이디 수정

    memcached-session-manager > core > StorageKeyFormat.java

    public static StorageKeyFormat of(final String config, final String host, final String context, final String webappVersion) {
      if(config == null || config.trim().isEmpty())
        return EMPTY;

      final String[] tokens = config.split(",");

      final StringBuilder sb = new StringBuilder();
      for (int i = 0; i < tokens.length; i++) {
        final String value = PrefixTokenFactory.parse(tokens[i], host, context, webappVersion);
        if(value != null && !value.trim().isEmpty()) {
                if(sb.length() > 0)
                    sb.append(STORAGE_TOKEN_SEP);
                  sb.append(value);
        }
      }
      
      //final String prefix = sb.length() == 0 ? null : sb.append(STORAGE_KEY_SEP).toString();
      final String prefix =  ":1:django.contrib.sessions.cache"+sb.toString();
          return new StorageKeyFormat(prefix, config);
    }
    
## 세션 쿠키 아이디 소문자

    memcached-session-manager > tomcat7 > MemcachedBackupSessionManager.java

    @Override
    public String generateSessionId() {
        return _msm.newSessionId( super.generateSessionId().toLowerCase() );
    }

## 멤캐시 설정

    Servers > context.xml

     <Manager className='de.javakaffee.web.msm.MemcachedBackupSessionManager'
              memcachedNodes='192.168.33.75:11211'
              requestUriIgnorePattern='.*\.(ico|png|gif|jpg|css|js)$'  />
              
## 쿠키 키 설정

    Servers > server.xml

    <Context sessionCookieName="sessionid"/>
              
## 세션 만료시간 설정

    Servers > web.xml

    <session-config>
            <session-timeout>30</session-timeout>
            <cookie-config>
                <max-age>31536000</max-age>  <!-- 60 * 60 * 24 * 365 -->
          </cookie-config>
    </session-config>
              
## 멤캐시 serialize 설정

    TranscoderService.java > serialize

    public byte[] serialize( final MemcachedBackupSession session, final byte[] attributesData ) {

      /** 기존 session serialize **/
      final byte[] sessionData = serializeSessionFields( session );
        final byte[] sessionByte = new byte[ sessionData.length + attributesData.length ];
        System.arraycopy( sessionData, 0, sessionByte, 0, sessionData.length );
        System.arraycopy( attributesData, 0, sessionByte, sessionData.length, attributesData.length );

        byte[] result = null;

    Tab tab = new Tab();

    String sName = "";
    /** 장고에서 읽을 수 있도록 session에 들어있는 내용을 풀어서 tab에 넣어준다. **/
    Enumeration attEnum = session.getAttributeNames(); 
    while(attEnum.hasMoreElements()) { 
         sName=(String)attEnum.nextElement(); 
         tab.put(sName, session.getAttribute(sName));
    } 

    tab.put("_auth_user_backend", "django.contrib.auth.backends.ModelBackend");
    tab.put("sessionByte", sessionByte);

    Pickler pickler = new Pickler(true);
    try
    {
      result = pickler.dumps(tab);
    } catch (PickleException e) {
      e.printStackTrace();
    } catch (IOException e) {
      e.printStackTrace();
    }

    return result;
    }
    
## 멤캐시 deserialize 설정

    TranscoderService.java > deserialize

    public MemcachedBackupSession deserialize( final byte[] data, final SessionManager manager ) {
      if ( data == null ) {
          return null;
      }

      try {

        Map<String, Object> map = new HashMap<>();

        Unpickler unpickler = new Unpickler();
        try {
          map = (Map<String, Object>) unpickler.loads(data);
    } catch (PickleException e) {
      e.printStackTrace();
    } catch (IOException e) {
      e.printStackTrace();
    }

        byte[] data1 = (byte[]) map.get("sessionByte");

          final DeserializationResult deserializationResult = deserializeSessionFields( data1, manager );
          final byte[] attributesData = deserializationResult.getAttributesData();
          final ConcurrentMap<String, Object> attributes = deserializeAttributes( attributesData );
          final MemcachedBackupSession session = deserializationResult.getSession();
          session.setAttributesInternal( attributes );
          session.setDataHashCode( Arrays.hashCode( attributesData ) );
          session.setManager( manager );
          session.doAfterDeserialization();
          return session;
      } catch( final InvalidVersionException e ) {
          LOG.info( "Got session data from memcached with an unsupported version: " + e.getVersion() );
          // for versioning probably there will be changes in the design,
          // with the first change and version 2 we'll see what we need
          return null;
      }
    }
    
## django

    memcachedStorageClient.java > ByteArrayTranscoder

    public static class ByteArrayTranscoder extends SerializingTranscoder {
    /**
     * Transcoder singleton instance.
     */
    public static final ByteArrayTranscoder INSTANCE = new ByteArrayTranscoder();
    private final TranscoderUtils tu=new TranscoderUtils(true);

    // General flags
    static final int SERIALIZED=1;
    static final int COMPRESSED=2;

    // Special flags for specially handled types.
    private static final int SPECIAL_MASK=0xff00;
    static final int SPECIAL_BOOLEAN=(1<<8);
    static final int SPECIAL_INT=(2<<8);
    static final int SPECIAL_LONG=(3<<8);
    static final int SPECIAL_DATE=(4<<8);
    static final int SPECIAL_BYTE=(5<<8);
    static final int SPECIAL_FLOAT=(6<<8);
    static final int SPECIAL_DOUBLE=(7<<8);
    static final int SPECIAL_BYTEARRAY=(8<<8);

    @Override
    public boolean asyncDecode(CachedData d) {
        return false;
    }

    /*
    @Override
    public byte[] decode(CachedData d) {
        return d.getData();
    }
    */
    public Object decode(CachedData d) {
    byte[] data=d.getData();
    Object rv=null;
    if((d.getFlags() & COMPRESSED) != 0) {
      data=decompress(d.getData());
    }
    int flags=d.getFlags() & SPECIAL_MASK;
    if((d.getFlags() & SERIALIZED) != 0 && data != null) {
      rv = data;
    } else if(flags != 0 && data != null) {
      switch(flags) {
        case SPECIAL_BOOLEAN:
          rv=Boolean.valueOf(tu.decodeBoolean(data));
          break;
        case SPECIAL_INT:
          rv=new Integer(tu.decodeInt(data));
          break;
        case SPECIAL_LONG:
          rv=new Long(tu.decodeLong(data));
          break;
        case SPECIAL_DATE:
          rv=new Date(tu.decodeLong(data));
          break;
        case SPECIAL_BYTE:
          rv=new Byte(tu.decodeByte(data));
          break;
        case SPECIAL_FLOAT:
          rv=new Float(Float.intBitsToFloat(tu.decodeInt(data)));
          break;
        case SPECIAL_DOUBLE:
    					rv=new Double(Double.longBitsToDouble(tu.decodeLong(data)));
    					break;
    				case SPECIAL_BYTEARRAY:
    					rv=data;
    					break;
    				default:
    					getLogger().warn("Undecodeable with flags %x", flags);
    			}
    		} else {
    			rv=decodeString(data);
    		}
    		return rv;
    	}
        
        /*
        @Override
        public CachedData encode(byte[] o) {
            return new CachedData(0, o, getMaxSize());
        }
        */
        public CachedData encode(Object o) {
    		byte[] b=null;
    		int flags=0;
    		if(o instanceof String) {
    			b=encodeString((String)o);
    		} else if(o instanceof Long) {
    			b=tu.encodeLong((Long)o);
    			flags |= SPECIAL_LONG;
    		} else if(o instanceof Integer) {
    			b=tu.encodeInt((Integer)o);
    			flags |= SPECIAL_INT;
    		} else if(o instanceof Boolean) {
    			b=tu.encodeBoolean((Boolean)o);
    			flags |= SPECIAL_BOOLEAN;
    		} else if(o instanceof Date) {
    			b=tu.encodeLong(((Date)o).getTime());
    			flags |= SPECIAL_DATE;
    		} else if(o instanceof Byte) {
    			b=tu.encodeByte((Byte)o);
    			flags |= SPECIAL_BYTE;
    		} else if(o instanceof Float) {
    			b=tu.encodeInt(Float.floatToRawIntBits((Float)o));
    			flags |= SPECIAL_FLOAT;
    		} else if(o instanceof Double) {
    			b=tu.encodeLong(Double.doubleToRawLongBits((Double)o));
    			flags |= SPECIAL_DOUBLE;
    		} else if(o instanceof byte[]) {
    			b=(byte[])o;
    			flags |= SERIALIZED;
    		} else {
    			b=serialize(o);
    			flags |= SERIALIZED;
    		}
    		assert b != null;
    		return new CachedData(flags, b, getMaxSize());
    	}
        
        @Override
        public int getMaxSize() {
            return CachedData.MAX_SIZE;
        }
    }


              
              
              
              
              
              
              
              
              
              
              
              
              
              
              
              
              
              
