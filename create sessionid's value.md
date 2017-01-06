## java base64

String target = testSHA256(v, s, u);
byte[] targetBytes = target.getBytes("UTF-8");

// Base64 인코딩 ///////////////////////////////////////////////////
Encoder encoder = Base64.getEncoder();

// Encoder#encode(byte[] src) :: 바이트배열로 반환
byte[] encodedBytes = encoder.encode(targetBytes);
System.out.println(new String(encodedBytes));

// Encoder#encodeToString(byte[] src) :: 문자열로 반환
String encodedString = encoder.encodeToString(targetBytes);
System.out.println(encodedString);

// Base64 디코딩 ///////////////////////////////////////////////////
Decoder decoder = Base64.getDecoder();

// Decoder#decode(bytes[] src) 
byte[] decodedBytes1 = decoder.decode(encodedBytes);
// Decoder#decode(String src)
byte[] decodedBytes2 = decoder.decode(encodedString);

// 디코딩한 문자열을 표시
String decodedString = new String(decodedBytes1, "UTF-8");
System.out.println(decodedString);

System.out.println(new String(decodedBytes2, "UTF-8"));



package kr.or.kotech.login.controller;

import java.io.UnsupportedEncodingException;
import java.security.InvalidKeyException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.security.SignatureException;
import java.util.Base64;
import java.util.Base64.Encoder;
import java.util.Formatter;

import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;

public class EdxCookieTest {

	public static void main(String[] args) throws Exception {
		
		/** safe_cookie_data = CURRENT_VERSION + session_Id + key_salt + signature **/
		
		String CURRENT_VERSION = "1";
		String SEPARATOR = "|";
		
		String session_Id = "s9ohn74f8p1nc150yafpi3gb5y4933la";
		
		String key_salt = get_random_string();
		
//		create signature = data_to_sign + key_salt 
		
//		Returns hash(version | session_id | user_id |)
		/** 1.data_to_sign **/
		String v = "1";
		String s = "s9ohn74f8p1nc150yafpi3gb5y4933la";
		String u = "";
		
		String data_to_sign = testSHA256(v, s, u);
		System.out.println("_____________data_to_sign: "+data_to_sign);
		
		/** 2. signature **/
		String encBase64 = compileBase64(data_to_sign);
		System.out.println("___________encBase64: "+encBase64);
		
		/** return TimestampSigner(key, salt=salt).sign(base64d) **/
		String timeValue = "";
		
		String sSep = ":";
		String sKey = "85920908f28904ed733fe576320db18cabd7b6cd";
		String sSalt = "8r7bEktxDLBCsigner";
		
		long epoch = System.currentTimeMillis()/1000;
		int intEpoch = (int)epoch;
		String base62Epoch = Base62.fromBase10(intEpoch);
		
		/** base64d : base62Epoch **/
		timeValue = encBase64 + sSep + base62Epoch;
		System.out.println("___________timeValue: " + timeValue);
		
		
		byte[] sha1_ad1 = null;
		sha1_ad1 = SHA1(key_salt+sKey);
		
		String hmac = calculateRFC2104HMAC("IjY3MmJhNzJkZTQ3MDQzMDBhNGExNTcwZWIwNDk5ZDRkZTQzNTZiOTFjZjdmY2MwYzhhOTVhNTRhMjk0MmJkYzEi:1cPO19", sha1_ad1);
		System.out.println(hmac);
		
		String encBase642= compileBase64(hmac);
		System.out.println("___________encBase642: "+encBase642);
		
		
		
		
		String tKey = "85920908f28904ed733fe576320db18cabd7b6cd";
		String tSalt = "8r7bEktxDLBCsigner";
		String tValue = "IjY3MmJhNzJkZTQ3MDQzMDBhNGExNTcwZWIwNDk5ZDRkZTQzNTZiOTFjZjdmY2MwYzhhOTVhNTRhMjk0MmJkYzEi:1cPO19";
		
	    try {
	    	
	        String secret = "secret";
  	        String message = "Message";

  	        /*
  	        message = bytes("Message").encode('utf-8')
    		secret = bytes("secret").encode('utf-8')

    		signature = base64.b64encode(hmac.new(secret, message, digestmod=hashlib.sha256).digest())
    		print(signature)
  	        */
  	        
	        Mac sha256_HMAC = Mac.getInstance("HmacSHA256");
	        SecretKeySpec secret_key = new SecretKeySpec(secret.getBytes(), "HmacSHA256");
	        sha256_HMAC.init(secret_key);

	        String hash = Base64.encodeBase64String(sha256_HMAC.doFinal(message.getBytes()));
	        System.out.println(hash);
	       }
	       catch (Exception e){
	        System.out.println("Error");
	       }
	    
	}
	
	/** 
	 * key_salt 생성
	 * length=12,
     * allowed_chars='abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789'
	**/
	public static String get_random_string(){
		/* number(0~9): 48~57, string(A~Z): 65~90, string(a~z): 97~122 */
		String key_salt = "";
		for(int i=0; i<12; i++) {
		  int rndVal = (int)(Math.random() * 62);
		  if(rndVal < 10) {
			  key_salt += rndVal;
		  } else if(rndVal > 35) {
			  key_salt += (char)(rndVal + 61);
		  } else {
			  key_salt += (char)(rndVal + 55);
		  }
		}
		return key_salt;
	}
	
	public static String testSHA256(String version, String sessionId, String user_id){
		
		if(user_id == null || user_id.equals("")){
			user_id = "None";
		}
				
		String str = version + "|" + sessionId + "|" + user_id + "|"; 
		String SHA = ""; 

		try{
			MessageDigest sh = MessageDigest.getInstance("SHA-256"); 
			sh.update(str.getBytes()); 
			byte byteData[] = sh.digest();
			StringBuffer sb = new StringBuffer(); 
			for(int i = 0 ; i < byteData.length ; i++){
				sb.append(Integer.toString((byteData[i]&0xff) + 0x100, 16).substring(1));
			}
			SHA = sb.toString();
			
		}catch(NoSuchAlgorithmException e){
			e.printStackTrace(); 
			SHA = null; 
		}
		return "\""+SHA+"\"";
	}
	
	/** base64 Encoding**/
	public static String compileBase64(String value) {
		
		String target = value;
        byte[] targetBytes;
        String encodedString = "";
    	
        try {
			targetBytes = target.getBytes("UTF-8");
			Encoder encoder = Base64.getEncoder();
			encodedString = encoder.encodeToString(targetBytes);
			
		} catch (UnsupportedEncodingException e) {
			e.printStackTrace();
		}	

		return encodedString;
	}
	
	
	
	
	
	
	
	
	
	
	
	
	
	 public static String convertToHex(byte[] data) { 
	        StringBuffer buf = new StringBuffer();
	        for (int i = 0; i < data.length; i++) { 
	            int halfbyte = (data[i] >>> 4) & 0x0F;
	            int two_halfs = 0;
	            do { 
	                if ((0 <= halfbyte) && (halfbyte <= 9)) 
	                    buf.append((char) ('0' + halfbyte));
	                else 
	                    buf.append((char) ('a' + (halfbyte - 10)));
	                halfbyte = data[i] & 0x0F;
	            } while(two_halfs++ < 1);
	        } 
	        return buf.toString();
	    } 
	 
	    public static byte[] SHA1(String text) throws NoSuchAlgorithmException, UnsupportedEncodingException  { 
	        MessageDigest md;
	        md = MessageDigest.getInstance("SHA-1");
	        byte[] sha1hash = new byte[40];
	        md.update(text.getBytes("iso-8859-1"), 0, text.length());
	        sha1hash = md.digest();
	        //return convertToHex(sha1hash);
	        return sha1hash;
	    } 
	    
	    private static final String HMAC_SHA1_ALGORITHM = "HmacSHA1";

		private static String toHexString(byte[] bytes) {
			Formatter formatter = new Formatter();
			
			for (byte b : bytes) {
				formatter.format("%02x", b);
			}

			return formatter.toString();
		}

		public static String calculateRFC2104HMAC(String data, String key)
			throws SignatureException, NoSuchAlgorithmException, InvalidKeyException
		{
			SecretKeySpec signingKey = new SecretKeySpec(key.getBytes(), HMAC_SHA1_ALGORITHM);
			Mac mac = Mac.getInstance(HMAC_SHA1_ALGORITHM);
			mac.init(signingKey);
			return toHexString(mac.doFinal(data.getBytes()));
		}

		public static String calculateRFC2104HMAC(String data, byte[] key)
			throws SignatureException, NoSuchAlgorithmException, InvalidKeyException
		{
			SecretKeySpec signingKey = new SecretKeySpec(key, HMAC_SHA1_ALGORITHM);
			Mac mac = Mac.getInstance(HMAC_SHA1_ALGORITHM);
			mac.init(signingKey);
			return toHexString(mac.doFinal(data.getBytes()));
		}
}


package kr.or.kotech.login.controller;

import com.picklingtools.pickle.Pickler;
import com.picklingtools.pythonesque.Tab;

public class EdxPickleTest {
	
	public static void main(String[] args) throws Exception {
		
		Tab t = new Tab();
		t.put("_auth_user_id", 5);
		t.put("_auth_user_hash", "0cf2e4a1d56e02e5accbde74ce9d5c59f354fdb3");
		t.put("_auth_user_backend", "ratelimitbackend.backends.RateLimitModelBackend");
		t.put("_language", "en");
		
        Pickler pickler = new Pickler(true);
        byte[] result =  pickler.dumps(t);
        System.out.println(result);
        
        
	}
	
}





package kr.or.kotech.login.controller;

import java.io.UnsupportedEncodingException;
import java.security.InvalidKeyException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.security.SignatureException;
import java.util.Formatter;

import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;

public class EdxUserHashTest {

	public static void main(String[] args) {

		byte[] sha1_ad1 = null;

		String key_salt = "django.contrib.auth.models.AbstractBaseUser.get_session_auth_hash";
		String secret = "85920908f28904ed733fe576320db18cabd7b6cd";
		String password = "pbkdf2_sha256$20000$g0utAuYI6ftZ$UbD5quSPcDpBrgeP6zHnRUZgul+Ls9ns3VF3IULQ6qM=";
		
		try {
			
			sha1_ad1 = SHA1(key_salt+secret);
			
		} catch (NoSuchAlgorithmException | UnsupportedEncodingException e) {
			e.printStackTrace();
		}
		
		String hmac = "";
		try {
		    hmac = calculateRFC2104HMAC(password, sha1_ad1);
	        } catch (Exception e) {
	        	e.printStackTrace();
	        }

		System.out.println("auth_user_hash: " + hmac);
		
	}
	
	public static String convertToHex(byte[] data) { 
        StringBuffer buf = new StringBuffer();
        for (int i = 0; i < data.length; i++) { 
            int halfbyte = (data[i] >>> 4) & 0x0F;
            int two_halfs = 0;
            do { 
                if ((0 <= halfbyte) && (halfbyte <= 9)) 
                    buf.append((char) ('0' + halfbyte));
                else 
                    buf.append((char) ('a' + (halfbyte - 10)));
                halfbyte = data[i] & 0x0F;
            } while(two_halfs++ < 1);
        } 
        return buf.toString();
    } 
 
    public static byte[] SHA1(String text) throws NoSuchAlgorithmException, UnsupportedEncodingException  { 
        MessageDigest md;
        md = MessageDigest.getInstance("SHA-1");
        byte[] sha1hash = new byte[40];
        md.update(text.getBytes("iso-8859-1"), 0, text.length());
        sha1hash = md.digest();
        return sha1hash;
    } 
    
    private static final String HMAC_SHA1_ALGORITHM = "HmacSHA1";

	private static String toHexString(byte[] bytes) {
		Formatter formatter = new Formatter();
		
		for (byte b : bytes) {
			formatter.format("%02x", b);
		}

		return formatter.toString();
	}

	public static String calculateRFC2104HMAC(String data, String key)
		throws SignatureException, NoSuchAlgorithmException, InvalidKeyException
	{
		SecretKeySpec signingKey = new SecretKeySpec(key.getBytes(), HMAC_SHA1_ALGORITHM);
		Mac mac = Mac.getInstance(HMAC_SHA1_ALGORITHM);
		mac.init(signingKey);
		return toHexString(mac.doFinal(data.getBytes()));
	}

	public static String calculateRFC2104HMAC(String data, byte[] key)
			throws SignatureException, NoSuchAlgorithmException, InvalidKeyException {
		SecretKeySpec signingKey = new SecretKeySpec(key, HMAC_SHA1_ALGORITHM);
		Mac mac = Mac.getInstance(HMAC_SHA1_ALGORITHM);
		mac.init(signingKey);
		return toHexString(mac.doFinal(data.getBytes()));
	}
}
