## Java Network

<img width="753" alt="image" src="https://user-images.githubusercontent.com/81945553/145668959-d4f1ea8e-2bb6-49ce-bd64-eb5586923fa5.png">

### Echo

- Server

  - ServerSocket (port) - waiting & accept()
  - 클라이언트의 요청이 와서 accept() 시 요청온 클라이언트용 소켓 추가 생성  
  - 송수신 버퍼 존재

- Client
  - Socket ( Socket -> ServerSocket 연결 요청) - socket(ip, port)
  - 송수신 버퍼 존재

- 결과 

  - <서버>
    Connected client : 127.0.0.1 : 49706
    Client's msg : sdas
    Client's msg : null

  - <클라이언트>
    Connection is completed
    Welcome to Echo server!!
    Message : sdas
    Echoed Message : sdas
    Message : BYE
    Disconnect 

``` java
import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.Scanner;

public class EchoServer {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        try {
            // 클라이언트의 접속을 처리하는 서버소켓 생성
            ServerSocket ss = new ServerSocket(8000);
            // 클라이언트 접속 대기
            Socket clients = ss.accept();
            System.out.println("Connected client : " + clients.getInetAddress()
                    .getHostAddress()+" : " + clients.getPort());

            //스트림 생성
            BufferedReader in = new BufferedReader(new InputStreamReader(clients.getInputStream()));
            PrintWriter out = new PrintWriter(new OutputStreamWriter(clients.getOutputStream()));
            // 데이터 송신
            out.println("Welcome to Echo server!!"); //송신 버퍼에 전달
            out.flush();

            while(true){
                String msg = in.readLine();
                System.out.println("Client's msg : " + msg);
                if(msg.equals("BYE") || msg == null) break;
                out.println(msg);
                out.flush();
            }
            clients.close();
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
            Socket s = new Socket("127.0.0.1", 8000);
            System.out.println("Connection is completed");

            BufferedReader in = new BufferedReader(new InputStreamReader(s.getInputStream()));
            PrintWriter out = new PrintWriter(new OutputStreamWriter(s.getOutputStream()));
            String data = in.readLine();
            System.out.println(data);
            while (true) {
                System.out.print("Message : ");
                String msg = sc.nextLine();
                if(msg.equals("BYE")) break;
                out.println(msg);
                out.flush();
                msg = in.readLine();
                System.out.println("Echoed Message : " + msg);
            }
            System.out.println("Disconnect ");
            s.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```



## Thread

- Thread란 하나의 프로세스 내부에서 독립적으로 실행되는 하나의 작업 단위

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
package network.thread;

import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.IOException;
import java.net.Socket;
import java.util.StringTokenizer;

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
        String received;
        while (true) {
            try {
                received = dis.readUTF();
                System.out.println("Message : "+received+" from " + name);
                if(received.equals("Bye")) break;
                if(!received.contains("#")) continue;
                StringTokenizer tokenizer = new StringTokenizer(received, "#");
                String who = tokenizer.nextToken();
                String msg = tokenizer.nextToken();
                for(ChatHandler c : ChatServer.clients) {
                    if (c.name.equals(who) && c.isSignin == true) {
                        c.dos.writeUTF(who +" >> " +msg);
                        break;
                    }
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        try {
            s.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```



```java
package network.thread;

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
        // 전송쓰레드 생성
        Thread sending = new Thread(new Runnable() { // 익명클래스 기법
            @Override
            public void run() {
                while (true) { // 데이터 송수신을 위한 내용
                    System.out.print("Msg(who#message) : ");
                    String msg = sc.nextLine();
                    try {
                        dos.writeUTF(msg);
                        if(msg.equals("Bye")) break;
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
                try {
                    s.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        });
        // 수신 쓰레드 생성
        Thread receiving = new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    try {
                        String msg = dis.readUTF();
                        if(msg == null) break; // 상대방이 접속 종료
                        System.out.println(msg);
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
                try {
                    s.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        });
        // 쓰레드 동작
        sending.start();
        receiving.start();
    }
}

```



## UDP

```java
package network.udp;


import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;

public class UDPEchoServer {
    public UDPEchoServer() {
        System.out.println("UDP EchoServer is started. Portno 9001");
        try {
            DatagramSocket socket = new DatagramSocket(9001);
            while (true) {
                // 데이터그램 패킷 수신
                byte[] message = new byte[1024];
                DatagramPacket packet = new DatagramPacket(message, message.length);
                socket.receive(packet);
                String data = new String(packet.getData());
                System.out.println("Received from : [" + data.trim() + "] \n" +
                        "From : " + packet.getAddress()); //원격지 주소 출력
                // 데이터그램 패킷 송신
                InetAddress address = packet.getAddress();
                int port = packet.getPort();
                byte[] sendMessage = data.getBytes();
                DatagramPacket sendPacket = new DatagramPacket(sendMessage, sendMessage.length, address, port);
                socket.send(sendPacket);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        new UDPEchoServer();
    }
}

```

```java
package network.udp;

import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.util.Scanner;

public class UDPEchoClient {

    public UDPEchoClient() {
        System.out.println("UDP EchoClient is started.");
        Scanner sc = new Scanner(System.in);
        try (DatagramSocket socket = new DatagramSocket()) {
            InetAddress address = InetAddress.getByName("localhost");
            byte[] message;
            while (true) {
                // 데이터 입력해서 데이터그램 패킷으로 만들고 전송
                System.out.println("Enter a message : ");
                String data = sc.nextLine();
                if(data.equalsIgnoreCase("quit")) break;
                message = data.getBytes(); // String -> byte[]
                DatagramPacket packet = new DatagramPacket(message, message.length, address, 9001);
                socket.send(packet); // 데이터그램 전송

                // 서버로부터 수신
                byte[] recvMessage = new byte[1024];
                DatagramPacket recvPacket = new DatagramPacket(recvMessage, recvMessage.length);
                socket.receive(recvPacket);
                data = new String(recvPacket.getData());
                System.out.println("Received : [" +data.trim() +" ] \n" +
                        " From : " + recvPacket.getSocketAddress());
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        new UDPEchoClient();
    }
}

```



## Java NIO

<img width="500" alt="image" src="https://user-images.githubusercontent.com/81945553/145674127-c519350b-d84a-4d0f-868b-442c5601792b.png">

- 기존의 쓰레드 방식은 쓰레드가 많아질수록 비효율적이지만, 셀렉터 기반 Channel + Buffer를 통해 해결가능 
- 스트림 방식인 서버소켓이 아닌 NIO에서 지원하는 서버소켓채널을 사용 

### Buffer 

- I / O 를 위한 메모리 공간 
- 버퍼 생성 (다이렉트 , 넌다이렉트) 
  - 다이렉트 : 운영체제가 메모리 할당 
    - 공간이 큼 - 메인메모리 전체에서 할당 받기 때문
    - 주로 읽기 / 쓰기가 많은 경우엔 다이렉트 버퍼가 유리
    - ByteBuffer.allocateDirect(20);
  - 넌다이렉트 : JVM이 메모리 할당 
    - 공간이 작음 - JVM에서 바로 처리하기 때문에 속도가 빠름
    - 버퍼를 생성, 삭제가 빈번하면 유리 
    - ByteBuffer.allocate(20);
- 버퍼 기록 / 읽기  
  - position : pointer와 같은 역할 
  - capacity : 전체 공간 , limit : 쓸 수 있는 공간 (처음엔 capacity와 동일) 
  - flip : 기록 / 읽기 변환 - 뒤집다라는 의미 , position을 0으로 초기화하고 limit은 현재 적힌 공간만큼으로 바뀜 
  - rewind : position을 0으로 초기화 
  - clear : position을 0으로 초기화 및 limit을 capacity로 초기화 (단, 그전에 읽은 값이 있다면 그건 유지)
  - mark : 현재 포인터에 체크 포인트, reset 하면 마지막 mark자리로 position 이동  
  - compact : 현재 까지 읽은 데이터를 지우고 앞으로 땡긴다. write상태로 바뀌며 , put 하면 현재포인터에 새 값이 채워짐
  - put (쓰기)
  - get (읽기)

```java
import java.nio.ByteBuffer;
import java.nio.CharBuffer;
import java.nio.IntBuffer;

public class JavaNIO {
    public static void printBuffer(ByteBuffer buffer) {
        // 버퍼에 있는 데이터 출력
        for (int i = 0; i < buffer.limit() - 1; i++) {
            System.out.print(buffer.get(i) + ", ");
        }
        System.out.println(buffer.get(buffer.limit() - 1));
        System.out.print("position : "+buffer.position()+", ");
        System.out.print("limit : " + buffer.limit()+", ");
        System.out.println("capacity : " + buffer.capacity());
    }
    public static void main(String[] args) {
        // 버퍼생성 - allocate 방식
        ByteBuffer buf = ByteBuffer.allocate(20); // 넌다이렉트
        ByteBuffer bufDir = ByteBuffer.allocateDirect(20); //다이렉트
        CharBuffer cbuf = CharBuffer.allocate(20);
        IntBuffer ibuf = IntBuffer.allocate(20);

        // 버퍼생성 - wrap 방식
        byte[] bytes = new byte[50];
        ByteBuffer buf2 = ByteBuffer.wrap(bytes);

        // 읽기/쓰기
        buf.put((byte) 7);
        buf.put((byte) 9);
        buf.put((byte) 13);
        buf.put((byte) 16);
        printBuffer(buf);
        buf.flip(); // 쓰기 -> 읽기, limit 갯수 변경, 포지션 초기화
        printBuffer(buf);
        buf.get(new byte[2]); // 2개 읽기 , position : 2
        printBuffer(buf);
        buf.mark(); // 현재 position 2에 mark기록
        buf.get(new byte[2]); // 2개 읽기, position : 4
        printBuffer(buf);
        buf.reset(); // mark위치로 position 이동
        printBuffer(buf);
        buf.rewind(); // position : 0 초기화
        printBuffer(buf);
        buf.get(new byte[2]); // position : 2
        buf.compact(); // 2개를 지우고 앞으로 땡김, 기록상태로 바뀜
        printBuffer(buf);
        buf.put((byte) 22);
        buf.put((byte) 22);
        buf.put((byte) 22);
        buf.put((byte) 22);
        printBuffer(buf);
        buf.clear();
        printBuffer(buf);
    }
}
```



### TCP (NIO)

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.Socket;
import java.nio.ByteBuffer;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.nio.charset.StandardCharsets;
import java.util.Date;

import static java.lang.Thread.sleep;

public class TimeServer {
    public static void main(String[] args) {
        System.out.println("Time Server is started.");
        try {
            ServerSocketChannel sschannel = ServerSocketChannel.open();
            // blocking 모드로 동작 (클라이언트 연결이 올 때까지 기다림)
            sschannel.configureBlocking(true);
            sschannel.socket().bind(new InetSocketAddress(5001)); // 포트 지정
            while (true) {
                System.out.println("Server is waiting for clients...");
                SocketChannel client = sschannel.accept();
                InetSocketAddress clientAddress = (InetSocketAddress) client.getRemoteAddress();
                System.out.println("Connected. " + clientAddress.getAddress() + " " + clientAddress.getPort());
                // Buffer creation
                ByteBuffer buf = ByteBuffer.allocate(64); // JVM
                for (int i = 0; i < 10; i++) {
                    String message = "Date: " + new Date(System.currentTimeMillis());
                    buf.clear(); // 이전에 입력한 데이터 삭제. position = 0, limit = capacity
                    buf.put(message.getBytes());
                    buf.flip(); // w -> r
                    while (buf.hasRemaining()) client.write(buf);
                    System.out.println("Sent: " + message);
                    sleep(1000);
                }
                client.close();

            }
        } catch (IOException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

``` java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.SocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SocketChannel;

import static java.lang.Thread.sleep;

public class TimeClient {
    public static void main(String[] args) {
        System.out.println("Time client is started");
        try {
            SocketAddress address = new InetSocketAddress("127.0.0.1", 5001);
            SocketChannel schannel = SocketChannel.open(address);
            System.out.println("Connected.");
            //Buffer creation
            ByteBuffer buffer = ByteBuffer.allocate(64);
            int bytes = schannel.read(buffer); // data receiving -> buffer
            System.out.println(bytes + " bytes are received.");
            while (bytes != -1) {
                buffer.flip(); // w -> r
//                while (buffer.hasRemaining()) System.out.print((char) buffer.get());
                byte[] data = new byte[buffer.limit()];
                buffer.get(data, 0, buffer.limit());
                String msg = new String(data);
                System.out.println(msg);
                sleep(1000);
                buffer.clear();
                bytes = schannel.read(buffer);
                System.out.println(bytes + " bytes are received.");
            }
        } catch (IOException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```



### UDP(NIO)

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.DatagramChannel;

public class UDPReceiver {
    public static void main(String[] args) throws IOException {
        InetSocketAddress address = new InetSocketAddress("127.0.0.1", 5005);
        DatagramChannel channel = DatagramChannel.open();
        channel.configureBlocking(true); // blocking mode
        channel.socket().bind(address);

        ByteBuffer buffer = ByteBuffer.allocate(64);
        while (true) {
            channel.receive(buffer);
            buffer.flip();
            byte[] data = new byte[buffer.limit()];
            buffer.get(data, 0, buffer.limit());
            String msg = new String(data);
            if(msg.equalsIgnoreCase("quit")) break;
            System.out.println(msg);
            buffer.clear();
        }
    }
}

```

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.DatagramChannel;
import java.util.Scanner;

public class UDPSender {
    private static Scanner sc = new Scanner(System.in);
    public static void main(String[] args) throws IOException {
        DatagramChannel channel = DatagramChannel.open();
        ByteBuffer buffer = ByteBuffer.allocate(64);
        InetSocketAddress address = new InetSocketAddress("127.0.0.1", 5005);
        while (true) {
            String msg;
            System.out.print("Message: ");
            msg = sc.nextLine();
            buffer.put(msg.getBytes());
            buffer.flip();
            channel.send(buffer, address);
            buffer.clear();
            if (msg.equalsIgnoreCase("quit")) break;
        }
    }
}
```

