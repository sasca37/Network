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

``` java
package network.echo;

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

