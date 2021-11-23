## Java Network

### Echo

- Server
  - ServerSocket (port) - waiting & accept()
  - 클라이언트의 요청이 와서 accept() 시 요청온 클라이언트용 소켓 추가 생성  
  - 송수신 버퍼 존재
- Client
  - Socket ( Socket -> ServerSocket 연결 요청) - socket(ip, port)
  - 송수신 버퍼 존재

``` java
import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;

public class EchoServer {
    public static void main(String[] args) {
        try {
            // 클라이언트의 접속을 처리하는 서버소켓 생성. 포트번호 지정.
            ServerSocket ss = new ServerSocket(8000);
            //클라이언트의 접속을 대기
            Socket client = ss.accept();
            System.out.println("Connected client : " + client.getInetAddress().getHostAddress() + " : "
                    + client.getPort());
            // 데이터 송수신을 위한 입출력 객체 생성
            BufferedReader in = new BufferedReader(new InputStreamReader(client.getInputStream()));
            PrintWriter out = new PrintWriter(new OutputStreamWriter(client.getOutputStream()));

            // 데이터 송신
            out.println("Welcome to Echo server!!"); // 송신 버퍼에 전달
            out.flush(); // 송신 버퍼의 모든 데이터를 전송

            while (true) {
                // 클라이언트 메시지 수신 후 그대로 전송
                String msg = in.readLine();
                System.out.println("Client's msg : " + msg);
                if(msg.equals("BYE") || msg == null) break;
                out.println(msg);
            }
            client.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```



```java
import java.io.*;
import java.net.Socket;
import java.util.Scanner;

public class EchoClient {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        try {
            // IP 주소, 포트번호로 서버에 접속 요청
            Socket s = new Socket("127.0.0.1", 8000);
            System.out.println("Connection is completed.");

            // 데이터 송수신을 위한 입출력 객체 생성
            BufferedReader in = new BufferedReader(new InputStreamReader(s.getInputStream()));
            PrintWriter out = new PrintWriter(new OutputStreamWriter(s.getOutputStream()));

            String data = in.readLine(); //수신
            System.out.println(data);
            while (true) {
                System.out.println("Message : ");
                String msg = sc.nextLine();
                if (msg.equals("BYE")) break;
                out.println(msg);
                out.flush();
                msg = in.readLine();
                System.out.println("Echoed Message : " + msg);
            }
            System.out.println("Disconnect...");
            s.close(); // 연결 종료 요청
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```



## Thread

- Thread 기반 서버 
  - 클라이언트와 서버 간 양방향 송수신을 위한 스트림 채널이 필요 
  - 스트림을 만든 후 서버는 **Thread를 통해 다시 wait 상태**로 간다.
  - 단, 서버의 과부하가 발생하므로 퍼포먼스 면에서 적합하지 않은 방법이다. 



```java
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.Vector;

public class ChatServer {
    static Vector<ChatHandler> clients = new Vector<>();
    public static void main(String[] args) throws IOException {
        ServerSocket ss = new ServerSocket(3005);
        while (true) {
            // 클라이언트 접속 대기
            System.out.println("Chat server is waiting ...");
            Socket s = ss.accept(); // block mode
            System.out.println("Client is connected : " + s);
            // 입출력 스트림 처리
            DataInputStream dis = new DataInputStream(s.getInputStream()); // 수신
            DataOutputStream dos = new DataOutputStream(s.getOutputStream()); // 송신

            dos.writeUTF("Welcome to a Chatting server");
            dos.writeUTF("Send your chatting id.");
            String name = dis.readUTF(); // 클라이언트 이름 수신
            ChatHandler handler = new ChatHandler(name, s, dis, dos);
            clients.add(handler);
            Thread thread = new Thread(handler);
            thread.start(); // handler 안에 있는 run 메서드가 동작
        }
    }
}
```

```java
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.net.Socket;

public class ChatHandler implements Runnable{
    private String name;
    private Socket s;
    private final DataInputStream dis;
    private final DataOutputStream dos;
    private boolean isSignin;

    public ChatHandler(String name, Socket s, DataInputStream dis, DataOutputStream dos) {
        this.name = name;
        this.s = s;
        this.dis = dis;
        this.dos = dos;
        isSignin = true;
    }

    @Override
    public void run() {
        System.out.println(name + " is chatting...");
    }
}

```

```java
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.IOException;
import java.net.InetAddress;
import java.net.Socket;
import java.util.Scanner;

public class ChatClient {
    final static int ServerPort = 3005;
    public static void main(String[] args) throws IOException {
        Scanner sc = new Scanner(System.in);
        InetAddress ipAddr = InetAddress.getByName("localhost");
        Socket s = new Socket(ipAddr, ServerPort);
        System.out.println("Connection OK!!");
        // 입출력 스트림 처리
        DataInputStream dis = new DataInputStream(s.getInputStream()); // 수신
        DataOutputStream dos = new DataOutputStream(s.getOutputStream()); // 송신

        String msg = dis.readUTF();
        System.out.println(msg);
        msg = dis.readUTF();
        System.out.println(msg);
        System.out.print("Name : ");
        String name = sc.nextLine();
        dos.writeUTF(name);
        while (true) {
            char q = sc.next().charAt(0);
            if(q == 'q') break;
        }
    }
}
```





## NIO

- Channel + Buffer로 구성 
- Thread를 대신하여 Selector 사용 

### Buffer

- 버퍼 생성 - 다이렉트 버퍼, 넌다이렉트 버퍼 
- 버퍼 기록 / 읽기 

