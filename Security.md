## Security

- 보안 정책 (Policy)
- H/W 보안 (칩 등)
- S/W 보안 (보안 솔루션)
  - 운영체제 
  - 네트워크 
  - 솔루션
  - 프로그래밍 (보안 모듈, 리버스 엔지니어링(역공학))



## Java Security 

- 메시지를 주고받을 때 (같은 프로그램, 즉 같은 알고리즘 사용)
  - 송신 : 암호화(Ciphertext + key)하여 송신  
  - 수신 : 받은 정보를 복호화하여 수신 
  - 공개키 : 암호화로 할때 사용한 키가 있어야 복호화가 가능한 방식  
    - DES : 1977년 IBM에 의해 개발, 현재 기술론 충분히 해킹 가능 (64비트 text, 54비트의 키,  16rounds를 통한 암호화)
    - AES(블록 알고리즘) : DES의 문제를 개선 - 128,192,256비트 등과 같은 가변적 비트길이, 키를 갖고 있어 복잡한 암호화된 결과물을 만들 수 있다. 128비트라면 블록안에 1칸(8비트 * 16을 구성하며, 1칸의 값을 16진수로 2자릿수변환), SubBytes 등 6번의 라운드를 거침 
    - 대칭키라고 불리며, 키를 수신자에게 보낼 때 가로채지 않도록 주의해야 한다.
  - 비밀키 : 암호화와 복호화의 키가 다른 경우
    - RSA : 암호화와 전자서명의 기능을 가진 알고리즘 
    - 한번 세션이 연결된 후 다시 연결할 때 새로운 키를 만들어야 한다.
    - 비대칭키 (public 키, private 키) : 서로다른 키를 갖고 암호화 복호화를 하기 때문에 소요시간이 길다. 송신자가 수신자에게 먼저 public(공개)키를 전달해주고, 송신자가 메시지에 private키로 암호화를하여 보내고 public 키로 복호화하여 수신, 다시 수신자는 public키로 암호화를 하고 송신자는 private키로 복호화하여 수신   

- Strong Type 
  - 타입이 정해진 메카니즘으로 중간에 바뀌지 않도록 함 

- 접근자 
  - 접근자 (private, protected 등의 캡슐화)를 통해 비정상적인 접근을 방지 

- Garbage Collection
  - 메모리 낭비 방지 (외부의 침입 방지 기능이 될 수 있음)

- Byte Code Verification
  - java -> class -> jvm 과정에서 보안 기능 향상

- Java Security Package
  - Java Cryptography Architecture (전자서명, 비밀키 등) - JCA
    - provider class 알고리즘 제공 - AES 등
    - Message Digest : Message -> SHA (Secure Hash Algorithm)

```java
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

public class Digest {
    public static void main(String[] args) throws NoSuchAlgorithmException {
        String msg = "Hello Crypto";
        String msg2 = "Hello Crypte";
        // Security Hash Algorithm 256비트 - SHA-256를 오브젝트로 가져온다.
        MessageDigest digest = MessageDigest.getInstance("SHA-256");
        digest.update(msg.getBytes()); // update()를 통해 암호화할 메시지 전달
        byte[] bytes = digest.digest(); // 암호화된 해쉬값 생성
        System.out.println(byteToHex(bytes));

        digest.update(msg2.getBytes()); // update()를 통해 암호화할 메시지 전달
        byte[] bytes2 = digest.digest(); // 암호화된 해쉬값 생성
        System.out.println(byteToHex(bytes2));
    }

    public static String byteToHex(byte[] bytes) {
        StringBuilder builder = new StringBuilder();
        // 2자릿수 16진수로 변경
        for(byte b : bytes) builder.append(String.format("%02x", b));
        return builder.toString();
    }
}

```



- Java SecureSocket Extension (SSL 등) - JSE

- 스니핑 
  - 네트워크 상에서 상대방의 패킷 교환을 훔쳐보는 것 (보통 허브에서 발생 )

### 카이사르 (시저) 암호 

- 입력받은 키 값만큼 값을 암호화 , 복호화 하는 기술 
- abc, 3 -> def (암호화) -> abc(복호화)

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Scanner;

public class CeaserCipher {
    static Scanner sc=new Scanner(System.in);
    static BufferedReader br = new BufferedReader(new InputStreamReader(System.in)); public
    static void main(String[] args) throws IOException {
        System.out.print("Enter any String: ");
        String str = br.readLine();
        System.out.print("\nEnter the Key: ");
        int key = sc.nextInt();
        String encrypted = encrypt(str, key);
        System.out.println("\nEncrypted String is: " + encrypted);
        String decrypted = decrypt(encrypted, key);
        System.out.println("\nDecrypted String is: " + decrypted);
        System.out.println("\n");
    }
    public static String encrypt(String str, int key) {
        String encrypted ="";
        for(int i = 0; i < str.length(); i++) {
            int c= str.charAt(i);
            if (Character.isUpperCase(c)) {
                c = c + (key % 26);
                if (c > 'Z') c = c - 26;
            }
            else if (Character.isLowerCase(c)) {
                c = c + (key % 26);
                if (c > 'z') c = c - 26;
            }
            encrypted += (char) c;
        }
        return encrypted;
    }
    public static String decrypt(String str, int key) {
        String decrypted = "";
        for(int i = 0; i < str.length(); i++) {
            int c = str.charAt(i);
            if (Character.isUpperCase(c)) {
                c = c - (key % 26);
                if (c < 'A') c = c + 26;
            }
            else if (Character.isLowerCase(c)) {
                c = c - (key % 26);
                if (c < 'a') c = c + 26;
            }
            decrypted += (char) c;
        }
        return decrypted;
    }
}
```



### DES & AES

- DES 와 AES는 같은 공개키 방식으로 DES -> AES로만 바꿔주면 AES로 동작한다. 
- <실행 결과>
  Cipher Text : jFqrsK5piL9KBdxv7BgObw==
  Plain Text : My name is John

```java
import javax.crypto.*;
import java.io.UnsupportedEncodingException;
import java.nio.charset.StandardCharsets;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.util.Base64;

public class Crypto {
    private static Cipher encryptor;
    private static Cipher decryptor;
    private static SecretKey key;
    public static void main(String[] args)
            throws NoSuchAlgorithmException, NoSuchPaddingException, InvalidKeyException, IllegalBlockSizeException, BadPaddingException, UnsupportedEncodingException {
        // DES 알고리즘 - generateKey()를 통한 키 생성
        key = KeyGenerator.getInstance("DES").generateKey();
        encryptor = Cipher.getInstance("DES");
        decryptor = Cipher.getInstance("DES");
        // 초기화
        encryptor.init(Cipher.ENCRYPT_MODE, key);
        decryptor.init(Cipher.DECRYPT_MODE, key);
        // 암호화 및 복호화
        String cipherText = encrypt("My name is John");
        System.out.println("Cipher Text : " + cipherText);
        String plainText = decrypt(cipherText);
        System.out.println("Plain Text : " + plainText);
    }

    private static String encrypt(String str)
            throws IllegalBlockSizeException, BadPaddingException {
        byte[] utf8 = str.getBytes(StandardCharsets.UTF_8);
        byte[] etext = encryptor.doFinal(utf8);
        //Base64 기법으로 16진수가 아닌 텍스트형태로 인코딩
        etext = Base64.getEncoder().encode(etext);
        return new String(etext);
    }

    private static String decrypt(String str) throws IllegalBlockSizeException, BadPaddingException, UnsupportedEncodingException {
        byte[] dtext = Base64.getDecoder().decode(str.getBytes());
        byte[] utf8 = decryptor.doFinal(dtext);
        return new String(utf8, "UTF8");
    }
}
```



### RSA

- 비밀키 방식으로 암호화와 복호화 키가 다른 경우 
- <실행 결과> 
  Cipher Text : ZVqA+0LNeLmg+oan60/9pE5rurJ8OGc4Exfo0uD4/ ... 
  Plain Text : My name is John

```java
import javax.crypto.*;
import java.io.UnsupportedEncodingException;
import java.nio.charset.StandardCharsets;
import java.security.InvalidKeyException;
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.NoSuchAlgorithmException;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;
import java.util.Base64;

public class RsaCrypto {
    private static Cipher encryptor;
    private static Cipher decryptor;
    private static SecretKey key;
    public static void main(String[] args)
            throws NoSuchAlgorithmException, NoSuchPaddingException, InvalidKeyException, IllegalBlockSizeException, BadPaddingException, UnsupportedEncodingException {
        KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance("RSA");
        KeyPair keyPair = keyPairGen.genKeyPair();
        RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();
        RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();
        encryptor = Cipher.getInstance("RSA");
        decryptor = Cipher.getInstance("RSA");

        // 초기화
        encryptor.init(Cipher.ENCRYPT_MODE, privateKey);
        decryptor.init(Cipher.DECRYPT_MODE, publicKey);
        // 암호화 및 복호화
        String cipherText = encrypt("My name is John");
        System.out.println("Cipher Text : " + cipherText);
        String plainText = decrypt(cipherText);
        System.out.println("Plain Text : " + plainText);
    }

    private static String encrypt(String str)
            throws IllegalBlockSizeException, BadPaddingException {
        byte[] utf8 = str.getBytes(StandardCharsets.UTF_8);
        byte[] etext = encryptor.doFinal(utf8);
        //Base64 기법으로 16진수가 아닌 텍스트형태로 인코딩
        etext = Base64.getEncoder().encode(etext);
        return new String(etext);
    }

    private static String decrypt(String str) throws IllegalBlockSizeException, BadPaddingException, UnsupportedEncodingException {
        byte[] dtext = Base64.getDecoder().decode(str.getBytes());
        byte[] utf8 = decryptor.doFinal(dtext);
        return new String(utf8, "UTF8");
    }
}
```



## SSL 

- Secure Sockets Layer로  네트워크 상  Application과 TCP 사이에서 동작하는 일종의 프로토콜 
- 불법적인 서버(사칭 등)에 접속해서 민감한 정보를 빼앗기는 부분을 방어 

### 제공 기능

- Authentication(인증) : 상호 간에 인증서(Certificate)로 인증  - CA(인증기관)을 통해 발급, java keytool (사설)
- Encryption : 메시지를 암호화(Cipher Text), 사전에 어떤 암호화 알고리즘을 쓸지 정해져 있음 (AES, RSA 등)
- Data Integrity : 전송한 데이터가 중간에 변형되지 않도록 보장 
  - Hash Function : SHA를 통해 얻게 된 Hash값을 데이터에 붙여서 전송 - hash값이 checksum 역할을 함 

### 동작과정 

- SSLServerSocket(서버), SSLSocket(클라이언트)을 사용 - SSL Process(Handshake)
  - Cipher Suite Negotiation (암호 협상) - 서버,클라이언트 간 SSL 버전 통일 
  - Authentication (Identify) - 인증, Certificate(인증서) 확인 과정 
  - Encryption algorithm - 알고리즘 결정 
- A가 B에게 public key를 보낼 때 certificate도 같이 보낸다. -> A가 B한테 비밀키를 암호화해서 보낸다. B는 public 키로 복호화해서 수신 -> B는 public key로 다시 암호화해서 A에게 보낸다. 

```java
import javax.net.ssl.HttpsURLConnection;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.URL;
import java.net.URLConnection;
import java.security.cert.Certificate;

public class HttpSecure {
    // URL 정보 확인 
    public static void getUrlInfo() throws IOException {
        URL url = new URL("https://www.oracle.com");
        BufferedReader reader = new BufferedReader(new InputStreamReader(url.openStream()));
        String line;
        while ((line = reader.readLine()) != null) {
            System.out.println(line);
        }
        reader.close();
    }
		// 인증서 확인 
    public static void getCertificate() throws IOException{
        URL url = new URL("https://www.oracle.com");
        URLConnection con = url.openConnection();
        HttpsURLConnection scon = (HttpsURLConnection) con; //보안 컨넥션으로 변경
        scon.connect(); //url에 ssl 기반 접속
        Certificate[] certs = scon.getServerCertificates();
        System.out.println("Server Certificate : " + certs[0].toString());
    }

    public static void main(String[] args) throws IOException {
//        getUrlInfo();
        getCertificate();
    }
}
```



## SSL Socket 통신 

- SSLServerSocket과 SSLSocket이 있어야 보안 통신이 가능하다.

  - 컨넥션이 되면  Cipher Suite, Authentication, Ecryption Algorithm 과정을 거친다. 

  - 서버는 CA(인증 기관)을 통해 인증서를 얻는다. -> 클라이언트는 서버에 Handshaking을 통해 CA로 발급받은 정상적인 인증서인지 확인 (잘못된 CA라면 Handshaking 실패)

- 서버 

  - SSLServerSocketFactory를 통해 SSLServerSocket이 생성 

- 클라이언트 

  - SSLSocketFactory를 통해 SSLSocket이 생성 

- SSL통신 테스트할 때 keytool(자바 사설 인증서)을 이용해 인증서를 만든 후 서버를 동작하는 방식 (인증서가 없으므로)

   ```java
   import javax.net.ssl.SSLSocket;
   import javax.net.ssl.SSLSocketFactory;
   import java.io.*;
   
   public class SSLClient {
       public static void main(String[] args) throws IOException {
           //String host = "www.oracle.com";
           String host = "127.0.0.1";
           SSLSocketFactory factory = (SSLSocketFactory) SSLSocketFactory.getDefault();
           //SSLSocket socket = (SSLSocket) factory.createSocket(host, 443);
           SSLSocket socket = (SSLSocket) factory.createSocket(host, 8089);
           socket.startHandshake(); // SSL Handshake
           PrintWriter out = new PrintWriter(new BufferedWriter(new OutputStreamWriter(socket.getOutputStream())));
           out.println("GET http://"+host+"/index.html HTTP/1.1"); //request 문장
           out.println();
           out.flush();
   
           BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
           String line;
           while ((line = in.readLine()) != null) {
               System.out.println(line);
           }
   
           in.close();
           out.close();
           socket.close();
       }
   }
   ```

### Java Keytool

- CA에서 인증받지못하므로 자바에서 제공하는 keytool을 통해 사설 인증서 생성
- java 파일 -> keytool -> server.jks 라는 이름으로 keytool 생성 

```java
import javax.net.ssl.*;
import java.io.*;
import java.security.KeyStore;

public class SSLServerKey {
    public static void main(String[] args) throws Exception {
        int port = 8089;
        String clientAuth = "No";
        System.out.println("USAGE: java SslServerCmd [port [clientAuth]]");
        if (args.length >= 1) port = Integer.parseInt(args[0]);
        if (args.length >= 2) clientAuth = args[1];

        KeyStore ks = KeyStore.getInstance("JKS"); // 자바 키스토어
        ks.load(new FileInputStream("server.jks"), "summer".toCharArray());

        KeyManagerFactory kmf = KeyManagerFactory.getInstance("SunX509");
        kmf.init(ks, "summer".toCharArray());

        SSLContext ctx = SSLContext.getInstance("TLS");
        ctx.init(kmf.getKeyManagers(), null, null);

        SSLServerSocketFactory ssf = ctx.getServerSocketFactory();
        SSLServerSocket ss = (SSLServerSocket)ssf.createServerSocket(port);
        if (clientAuth.equals("Yes")) ss.setNeedClientAuth(true);

        System.out.println("Listening: port="+port+", clientAuth="+clientAuth);
        SSLSocket socket = (SSLSocket) ss.accept();

        BufferedReader in = new BufferedReader(new InputStreamReader(
                socket.getInputStream()));
        String line = in.readLine();
        while (line.length()>0) {
            System.out.println(line);
            line = in.readLine();
        }

        PrintWriter out = new PrintWriter(new BufferedWriter(
                new OutputStreamWriter(socket.getOutputStream())));
        out.println("HTTP/1.0 200 OK");
        out.println("Content-Type: text/html");
        out.println("Content-Length: 40"); // including \r\n
        out.println();
        out.println("<html><body>Hello there!</body></html>");
        out.flush();

        out.close();
        in.close();
        socket.close();
    }
}
```

