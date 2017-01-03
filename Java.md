### Map 반복문
  
         1. 
         Iterator<String> keys = map.keySet().iterator();
         while( keys.hasNext() ){
             String key = keys.next();
             System.out.println( String.format("키 : %s, 값 : %s", key, map.get(key)) );
         }
         
        2.
        for( Map.Entry<String, String> elem : map.entrySet() ){
            System.out.println( String.format("키 : %s, 값 : %s", elem.getKey(), elem.getValue()) );
        }
         
        3.
        for( String key : map.keySet() ){
            System.out.println( String.format("키 : %s, 값 : %s", key, map.get(key)) );
        }

### 모든 세션키 가져오기

    Enumeration attEnum = session.getAttributeNames(); 
    while(attEnum.hasMoreElements()) { 
         sName=(String)attEnum.nextElement(); 
         tab.put(sName, session.getAttribute(sName));
    } 



