# 一、概述

Java 的 I/O 大概可以分成以下几类：

- 磁盘操作：File
- 字节操作：InputStream 和 OutputStream
- 字符操作：Reader 和 Writer
- 对象操作：Serializable
- 网络操作：Socket
- 新的输入/输出：NIO

# 二、磁盘操作

File 类可以用于表示文件和目录的信息，但是它不表示文件的内容。

递归地列出一个目录下所有文件：

	public static void listAllFiles(File dir) {
	    if (dir == null || !dir.exists()) {
	        return;
	    }
	    if (dir.isFile()) {
	        System.out.println(dir.getName());
	        return;
	    }
	    for (File file : dir.listFiles()) {
	        listAllFiles(file);
	    }
	}

# 三、字节操作

## 实现文件复制

	public static void copyFile(String src, String dist) throws IOException {
	    FileInputStream in = new FileInputStream(src);
	    FileOutputStream out = new FileOutputStream(dist);
	
	    byte[] buffer = new byte[20 * 1024];
	    int cnt;
	
	    // read() 最多读取 buffer.length 个字节
	    // 返回的是实际读取的个数
	    // 返回 -1 的时候表示读到 eof，即文件尾
	    while ((cnt = in.read(buffer, 0, buffer.length)) != -1) {
	        out.write(buffer, 0, cnt);
	    }
	
	    in.close();
	    out.close();
	}

## 装饰者模式

Java I/O 使用了装饰者模式来实现。以 InputStream 为例，

- InputStream 是抽象组件；
- FileInputStream 是 InputStream 的子类，属于具体组件，提供了字节流的输入操作；
- FilterInputStream 属于抽象装饰者，装饰者用于装饰组件，为组件提供额外的功能。例如 BufferedInputStream 为 FileInputStream 提供缓存的功能。

![](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/9709694b-db05-4cce-8d2f-1c8b09f4d921.png)

实例化一个具有缓存功能的字节流对象时，只需要在 FileInputStream 对象上再套一层 BufferedInputStream 对象即可。

	FileInputStream fileInputStream = new FileInputStream(filePath);
	BufferedInputStream bufferedInputStream = new BufferedInputStream(fileInputStream);

DataInputStream 装饰者提供了对更多数据类型进行输入的操作，比如 int、double 等基本类型。

# 四、字符操作 #

## 编码与解码 ##

编码就是把字符转换为字节，而解码是把字节重新组合成字符。

如果编码和解码过程使用不同的编码方式那么就出现了乱码。

- GBK 编码中，中文字符占 2 个字节，英文字符占 1 个字节；
- UTF-8 编码中，中文字符占 3 个字节，英文字符占 1 个字节；
- UTF-16be 编码中，中文字符和英文字符都占 2 个字节。

UTF-16be 中的 be 指的是 Big Endian，也就是大端。相应地也有 UTF-16le，le 指的是 Little Endian，也就是小端。

Java 的内存编码使用双字节编码 UTF-16be，这不是指 Java 只支持这一种编码方式，而是说 char 这种类型使用 UTF-16be 进行编码。char 类型占 16 位，也就是两个字节，Java 使用这种双字节编码是为了让一个中文或者一个英文都能使用一个 char 来存储。

## String 的编码方式 ##

String 可以看成一个字符序列，可以指定一个编码方式将它编码为字节序列，也可以指定一个编码方式将一个字节序列解码为 String。

	String str1 = "中文";
	byte[] bytes = str1.getBytes("UTF-8");
	String str2 = new String(bytes, "UTF-8");
	System.out.println(str2);

在调用无参数 getBytes() 方法时，默认的编码方式不是 UTF-16be。双字节编码的好处是可以使用一个 char 存储中文和英文，而将 String 转为 bytes[] 字节数组就不再需要这个好处，因此也就不再需要双字节编码。getBytes() 的默认编码方式与平台有关，一般为 UTF-8。

	byte[] bytes = str1.getBytes();

## Reader 与 Writer ##

不管是磁盘还是网络传输，最小的存储单元都是字节，而不是字符。但是在程序中操作的通常是字符形式的数据，因此需要提供对字符进行操作的方法。

- InputStreamReader 实现从字节流解码成字符流；
- OutputStreamWriter 实现字符流编码成为字节流。

这里用到了适配器模式

## 实现逐行输出文本文件的内容

	public static void readFileContent(String filePath) throws IOException {
	
	    FileReader fileReader = new FileReader(filePath);
	    BufferedReader bufferedReader = new BufferedReader(fileReader);
	
	    String line;
	    while ((line = bufferedReader.readLine()) != null) {
	        System.out.println(line);
	    }
	
	    // 装饰者模式使得 BufferedReader 组合了一个 Reader 对象
	    // 在调用 BufferedReader 的 close() 方法时会去调用 Reader 的 close() 方法
	    // 因此只要一个 close() 调用即可
	    bufferedReader.close();
	}

# 五、对象操作 #

## 序列化 ##

序列化就是将一个对象转换成字节序列，方便存储和传输。

- 序列化：ObjectOutputStream.writeObject()
- 反序列化：ObjectInputStream.readObject()

不会对静态变量进行序列化，因为序列化只是保存对象的状态，静态变量属于类的状态。

## Serializable ##

序列化的类需要实现 Serializable 接口，它只是一个标准，没有任何方法需要实现，但是如果不去实现它的话而进行序列化，会抛出异常。

	public static void main(String[] args) throws IOException, ClassNotFoundException {
	
	    A a1 = new A(123, "abc");
	    String objectFile = "file/a1";
	
	    ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream(objectFile));
	    objectOutputStream.writeObject(a1);
	    objectOutputStream.close();
	
	    ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream(objectFile));
	    A a2 = (A) objectInputStream.readObject();
	    objectInputStream.close();
	    System.out.println(a2);
	}
	
	private static class A implements Serializable {
	
	    private int x;
	    private String y;
	
	    A(int x, String y) {
	        this.x = x;
	        this.y = y;
	    }
	
	    @Override
	    public String toString() {
	        return "x = " + x + "  " + "y = " + y;
	    }
	}

## transient ##

transient 关键字可以使一些属性不会被序列化。

ArrayList 中存储数据的数组 elementData 是用 transient 修饰的，因为这个数组是动态扩展的，并不是所有的空间都被使用，因此就不需要所有的内容都被序列化。通过重写序列化和反序列化方法，使得可以只序列化数组中有内容的那部分数据。

	private transient Object[] elementData;

# 六、网络操作 #

Java 中的网络支持：

- InetAddress：用于表示网络上的硬件资源，即 IP 地址；
- URL：统一资源定位符；
- Sockets：使用 TCP 协议实现网络通信；
- Datagram：使用 UDP 协议实现网络通信。

## InetAddress ##

没有公有的构造函数，只能通过静态方法来创建实例。

	InetAddress.getByName(String host);
	InetAddress.getByAddress(byte[] address);

## URL ##

可以直接从 URL 中读取字节流数据。

	public static void main(String[] args) throws IOException {
	
	    URL url = new URL("http://www.baidu.com");
	
	    /* 字节流 */
	    InputStream is = url.openStream();
	
	    /* 字符流 */
	    InputStreamReader isr = new InputStreamReader(is, "utf-8");
	
	    /* 提供缓存功能 */
	    BufferedReader br = new BufferedReader(isr);
	
	    String line;
	    while ((line = br.readLine()) != null) {
	        System.out.println(line);
	    }
	
	    br.close();
	}

## Sockets

- ServerSocket：服务器端类
- Socket：客户端类
- 服务器和客户端通过 InputStream 和 OutputStream 进行输入输出。

![](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/1e6affc4-18e5-4596-96ef-fb84c63bf88a.png)

客户端

	public class Client {
	
	    public static void main(String[] args) throws IOException {
	        //端口
	        Integer port = 9526;
	        //ip
	        String ip = "127.0.0.1";
	        //创建socket对象
	        Socket socket = new Socket(ip, port);
	        //准备输入输出流,包装
	        DataInputStream dataInputStream = new DataInputStream(socket.getInputStream());
	        DataOutputStream dataOutputStream = new DataOutputStream(socket.getOutputStream());
	
	        //发送消息
	        dataOutputStream.writeUTF("zzy");
	
	        //接受消息
	        String msg = dataInputStream.readUTF();
	        System.out.println("服务端："+msg);
	
	        //释放资源
	        dataInputStream.close();
	        dataOutputStream.close();
	        socket.close();
	
	    }
	}

服务端

	public class Server {
	
	    public static void main(String[] args) throws IOException {
	        //指定端口
	        Integer port = 9526;
	        //注册服务端的socket对象
	        ServerSocket serverSocket = new ServerSocket(port);
	
	        System.out.println("socket服务注册成功");
	        System.out.println("等待客户端连接……");
	
	        while (true){
	            //监听-阻塞
	            Socket accept = serverSocket.accept();
	            //创建并启动线程
	            new ServerThread(accept).start();
	        }
	    }
	
	}

服务端线程类

	public class ServerThread extends Thread{
	
	    /**
	     * socket连接对象
	     */
	    private Socket socket;
	
	    private DataOutputStream dataOutputStream;
	
	    private DataInputStream dataInputStream;
	
	    public ServerThread(Socket socket){
	        this.socket = socket;
	    }
	
	
	    @Override
	    public void run() {
	        try {
	            //获取客户端信息
	            InetAddress inetAddress = socket.getInetAddress();
	            //ip地址
	            String ip = inetAddress.getHostAddress();
	            System.out.println("客户端："+ip+"连接成功");
	            //输入输出流
	            dataInputStream = new DataInputStream(socket.getInputStream());
	            dataOutputStream = new DataOutputStream(socket.getOutputStream());
	            //接受客户端消息
	            String msg = dataInputStream.readUTF();
	            System.out.println("客户端："+msg);
	
	            //向客户端发送消息
	            dataOutputStream.writeUTF("收到");
	
	
	        }catch (IOException e){
	            e.printStackTrace();
	        }finally {
	            closeResources(dataInputStream, dataOutputStream, socket);
	        }
	
	    }
	
	    private void closeResources(DataInputStream dataInputStream, DataOutputStream dataOutputStream, Socket socket){
	
	        try {
	            if(dataInputStream != null){
	                dataInputStream.close();
	            }
	            if(dataOutputStream != null){
	                dataOutputStream.close();
	            }
	            if(socket != null){
	                socket.close();
	            }
	        } catch (IOException e) {
	            e.printStackTrace();
	        }
	    }
	}

## Datagram

- DatagramSocket：通信类
- DatagramPacket：数据包类

# 七、NIO #

新的输入/输出 (NIO) 库是在 JDK 1.4 中引入的，弥补了原来的 I/O 的不足，提供了高速的、面向块的 I/O。

## 流与块 ##

I/O 与 NIO 最重要的区别是数据打包和传输的方式，I/O 以流的方式处理数据，而 NIO 以块的方式处理数据。

面向流的 I/O 一次处理一个字节数据：一个输入流产生一个字节数据，一个输出流消费一个字节数据。为流式数据创建过滤器非常容易，链接几个过滤器，以便每个过滤器只负责复杂处理机制的一部分。不利的一面是，面向流的 I/O 通常相当慢。

面向块的 I/O 一次处理一个数据块，按块处理数据比按流处理数据要快得多。但是面向块的 I/O 缺少一些面向流的 I/O 所具有的优雅性和简单性。

I/O 包和 NIO 已经很好地集成了，java.io.* 已经以 NIO 为基础重新实现了，所以现在它可以利用 NIO 的一些特性。例如，java.io.* 包中的一些类包含以块的形式读写数据的方法，这使得即使在面向流的系统中，处理速度也会更快。