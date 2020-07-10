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

将文件内容读取到缓冲区

将缓冲区内容写入到文件中

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

I/O 包和 NIO 已经很好地集成了，java.io.* 已经以 NIO 为基础重新实现了，所以现在它可以利用 NIO 的一些特性。例如，java.io. 包中的一些类包含以块的形式读写数据的方法，这使得即使在面向流的系统中，处理速度也会更快。

## 通道与缓冲区 ##

### 1. 通道 ###

通道 Channel 是对原 I/O 包中的流的模拟，可以通过它读取和写入数据。

通道与流的不同之处在于，流只能在一个方向上移动(一个流必须是 InputStream 或者 OutputStream 的子类)，而通道是双向的，可以用于读、写或者同时用于读写。

通道包括以下类型：

- FileChannel：从文件中读写数据；
- DatagramChannel：通过 UDP 读写网络中数据；
- SocketChannel：通过 TCP 读写网络中数据；
- ServerSocketChannel：可以监听新进来的 TCP 连接，对每一个新进来的连接都会创建一个 SocketChannel。

### 2. 缓冲区 ###

发送给一个通道的所有数据都必须首先放到缓冲区中，同样地，从通道中读取的任何数据都要先读到缓冲区中。也就是说，不会直接对通道进行读写数据，而是要先经过缓冲区。

缓冲区实质上是一个数组，但它不仅仅是一个数组。缓冲区提供了对数据的结构化访问，而且还可以跟踪系统的读/写进程。

缓冲区包括以下类型：

- ByteBuffer
- CharBuffer
- ShortBuffer
- IntBuffer
- LongBuffer
- FloatBuffer
- DoubleBuffer

## 缓冲区状态变量 ##

- capacity：最大容量；
- position：当前已经读写的字节数；
- limit：还可以读写的字节数。

状态变量的改变过程举例：

① 新建一个大小为 8 个字节的缓冲区，此时 position 为 0，而 limit = capacity = 8。capacity 变量不会改变，下面的讨论会忽略它。

![](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/1bea398f-17a7-4f67-a90b-9e2d243eaa9a.png)

② 从输入通道中读取 5 个字节数据写入缓冲区中，此时 position 为 5，limit 保持不变。

![](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/80804f52-8815-4096-b506-48eef3eed5c6.png)

③ 在将缓冲区的数据写到输出通道之前，需要先调用 flip() 方法，这个方法将 limit 设置为当前 position，并将 position 设置为 0。

![](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/952e06bd-5a65-4cab-82e4-dd1536462f38.png)

④ 从缓冲区中取 4 个字节到输出缓冲中，此时 position 设为 4。

![](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/b5bdcbe2-b958-4aef-9151-6ad963cb28b4.png)

⑤ 最后需要调用 clear() 方法来清空缓冲区，此时 position 和 limit 都被设置为最初位置。

![](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/67bf5487-c45d-49b6-b9c0-a058d8c68902.png)

## 文件 NIO 实例 ##

	public static void fastCopy(String src, String dist) throws IOException {
	
	    /* 获得源文件的输入字节流 */
	    FileInputStream fin = new FileInputStream(src);
	
	    /* 获取输入字节流的文件通道 */
	    FileChannel fcin = fin.getChannel();
	
	    /* 获取目标文件的输出字节流 */
	    FileOutputStream fout = new FileOutputStream(dist);
	
	    /* 获取输出字节流的文件通道 */
	    FileChannel fcout = fout.getChannel();
	
	    /* 为缓冲区分配 1024 个字节 */
	    ByteBuffer buffer = ByteBuffer.allocateDirect(1024);
	
	    while (true) {
	
	        /* 从输入通道中读取数据到缓冲区中 */
	        int r = fcin.read(buffer);
	
	        /* read() 返回 -1 表示 EOF */
	        if (r == -1) {
	            break;
	        }
	
	        /* 切换读写 */
	        buffer.flip();
	
	        /* 把缓冲区的内容写入输出文件中 */
	        fcout.write(buffer);
	
	        /* 清空缓冲区 */
	        buffer.clear();
	    }
	}

## 选择器 ##

NIO 常常被叫做非阻塞 IO，主要是因为 NIO 在网络通信中的非阻塞特性被广泛使用。

NIO 实现了 IO 多路复用中的 Reactor 模型，一个线程 Thread 使用一个选择器 Selector 通过轮询的方式去监听多个通道 Channel 上的事件，从而让一个线程就可以处理多个事件。

通过配置监听的通道 Channel 为非阻塞，那么当 Channel 上的 IO 事件还未到达时，就不会进入阻塞状态一直等待，而是继续轮询其它 Channel，找到 IO 事件已经到达的 Channel 执行。

因为创建和切换线程的开销很大，因此使用一个线程来处理多个事件而不是一个线程处理一个事件，对于 IO 密集型的应用具有很好地性能。

应该注意的是，只有套接字 Channel 才能配置为非阻塞，而 FileChannel 不能，为 FileChannel 配置非阻塞也没有意义。

![](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/093f9e57-429c-413a-83ee-c689ba596cef.png)

### 1. 创建选择器 ###

	Selector selector = Selector.open();

### 2. 将通道注册到选择器上 ###

	ServerSocketChannel ssChannel = ServerSocketChannel.open();
	ssChannel.configureBlocking(false);
	ssChannel.register(selector, SelectionKey.OP_ACCEPT);

ServerSocketChannel：可以监听新进来的 TCP 连接，对每一个新进来的连接都会创建一个 SocketChannel。

通道必须配置为非阻塞模式，否则使用选择器就没有任何意义了，因为如果通道在某个事件上被阻塞，那么服务器就不能响应其它事件，必须等待这个事件处理完毕才能去处理其它事件，显然这和选择器的作用背道而驰。

在将通道注册到选择器上时，还需要指定要注册的具体事件，主要有以下几类：

- SelectionKey.OP_CONNECT
- SelectionKey.OP_ACCEPT
- SelectionKey.OP_READ
- SelectionKey.OP_WRITE

它们在 SelectionKey 的定义如下：

	public static final int OP_READ = 1 << 0;
	public static final int OP_WRITE = 1 << 2;
	public static final int OP_CONNECT = 1 << 3;
	public static final int OP_ACCEPT = 1 << 4;

可以看出每个事件可以被当成一个位域，从而组成事件集整数。例如：

	int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;


### 3. 监听事件 ###

	int num = selector.select();

使用 select() 来监听到达的事件，它会一直阻塞直到有至少一个事件到达。

### 4. 获取到达的事件 ###

	Set<SelectionKey> keys = selector.selectedKeys();
	Iterator<SelectionKey> keyIterator = keys.iterator();
	while (keyIterator.hasNext()) {
	    SelectionKey key = keyIterator.next();
	    if (key.isAcceptable()) {
	        // ...
	    } else if (key.isReadable()) {
	        // ...
	    }
	    keyIterator.remove();
	}

### 5. 事件循环 ###

因为一次 select() 调用不能处理完所有的事件，并且服务器端有可能需要一直监听事件，因此服务器端处理事件的代码一般会放在一个死循环内。

	while (true) {
	    int num = selector.select();
	    Set<SelectionKey> keys = selector.selectedKeys();
	    Iterator<SelectionKey> keyIterator = keys.iterator();
	    while (keyIterator.hasNext()) {
	        SelectionKey key = keyIterator.next();
	        if (key.isAcceptable()) {
	            // ...
	        } else if (key.isReadable()) {
	            // ...
	        }
	        keyIterator.remove();
	    }
	}

## 套接字 NIO 实例 ##

服务端

	public class NIOServer {
	
	    public static void main(String[] args) throws IOException {
	
	        //创建选择器
	        Selector selector = Selector.open();
	
	        //通道注册到选择器上
	        //ServerSocketChannel：可以监听新进来的 TCP 连接，对每一个新进来的连接都会创建一个 SocketChannel。
	        ServerSocketChannel ssChannel = ServerSocketChannel.open();
	        //通道配置为非阻塞模式
	        ssChannel.configureBlocking(false);
	        //指定要注册的具体事件
	        ssChannel.register(selector, SelectionKey.OP_ACCEPT);
	
	        //创建ServerSocket
	        ServerSocket serverSocket = ssChannel.socket();
	        InetSocketAddress address = new InetSocketAddress("127.0.0.1", 8888);
	        //监听
	        serverSocket.bind(address);
	
	        //事件循环
	        while (true) {
	
	            //使用 select() 来监听到达的事件，它会一直阻塞直到有至少一个事件到达。
	            selector.select();
	
	            //获取到达的事件
	            Set<SelectionKey> keys = selector.selectedKeys();
	            Iterator<SelectionKey> keyIterator = keys.iterator();
	
	            while (keyIterator.hasNext()) {
	
	                SelectionKey key = keyIterator.next();
	
	                if (key.isAcceptable()) {
	
	                    ServerSocketChannel ssChannel1 = (ServerSocketChannel) key.channel();
	
	                    // 服务器会为每个新连接创建一个 SocketChannel
	                    SocketChannel sChannel = ssChannel1.accept();
	                    sChannel.configureBlocking(false);
	
	                    // 这个新连接主要用于从客户端读取数据
	                    sChannel.register(selector, SelectionKey.OP_READ);
	
	                } else if (key.isReadable()) {
	
	                    SocketChannel sChannel = (SocketChannel) key.channel();
	                    System.out.println(readDataFromSocketChannel(sChannel));
	                    sChannel.close();
	                }
	
	                keyIterator.remove();
	            }
	        }
	    }
	
	    private static String readDataFromSocketChannel(SocketChannel sChannel) throws IOException {
	
	        ByteBuffer buffer = ByteBuffer.allocate(1024);
	        StringBuilder data = new StringBuilder();
	
	        while (true) {
	
	            buffer.clear();
	            int n = sChannel.read(buffer);
	            if (n == -1) {
	                break;
	            }
	            buffer.flip();
	            int limit = buffer.limit();
	            char[] dst = new char[limit];
	            for (int i = 0; i < limit; i++) {
	                dst[i] = (char) buffer.get(i);
	            }
	            data.append(dst);
	            buffer.clear();
	        }
	        return data.toString();
	    }
	}

客户端

	public class NIOClient {
	
	    public static void main(String[] args) throws IOException {
	        Socket socket = new Socket("127.0.0.1", 8888);
	        OutputStream out = socket.getOutputStream();
	        String s = "hello world";
	        out.write(s.getBytes());
	        out.close();
	    }
	}

## 内存映射文件 ##

内存映射文件 I/O 是一种读和写文件数据的方法，它可以比常规的基于流或者基于通道的 I/O 快得多。

向内存映射文件写入可能是危险的，只是改变数组的单个元素这样的简单操作，就可能会直接修改磁盘上的文件。修改数据与将数据保存到磁盘是没有分开的。

下面代码行将文件的前 1024 个字节映射到内存中，map() 方法返回一个 MappedByteBuffer，它是 ByteBuffer 的子类。因此，可以像使用其他任何 ByteBuffer 一样使用新映射的缓冲区，操作系统会在需要时负责执行映射。

	MappedByteBuffer mbb = fc.map(FileChannel.MapMode.READ_WRITE, 0, 1024);

## 对比 ##

NIO 与普通 I/O 的区别主要有以下两点：

- NIO 是非阻塞的；
- NIO 面向块，I/O 面向流。

# 八、BIO,NIO,AIO 对比 #

## 1. “同步/异步 ”和 “阻塞/非阻塞” ##

同步/异步是从行为角度描述事物的，而阻塞和非阻塞描述的当前事物的状态（等待调用结果时的状态）。

### 同步/异步 ###

- 同步 ：两个同步任务相互依赖，并且一个任务必须以依赖于另一任务的某种方式执行。 比如在A->B事件模型中，你需要先完成 A 才能执行B。 再换句话说，同步调用种被调用者未处理完请求之前，调用不返回，调用者会一直等待结果的返回。
- 异步： 两个异步的任务完全独立的，一方的执行不需要等待另外一方的执行。再换句话说，异步调用种一调用就返回结果不需要等待结果返回，当结果返回的时候通过回调函数或者其他方式拿着结果再做相关事情。

### 阻塞/非阻塞 ###

- 阻塞： 阻塞就是发起一个请求，调用者一直等待请求结果返回，也就是当前线程会被挂起，无法从事其他任务，只有当条件就绪才能继续。
- 非阻塞： 非阻塞就是发起一个请求，调用者不用一直等着结果返回，可以先去干其他事情。

## 2. BIO (Blocking I/O) ##

同步阻塞I/O模式，数据的读取写入必须阻塞在一个线程内等待其完成。

采用 **BIO 通信模型** 的服务端，通常由一个独立的 Acceptor 线程负责监听客户端的连接。我们一般通过在while(true) 循环中服务端会调用 accept() 方法等待接收客户端的连接的方式监听请求，请求一旦接收到一个连接请求，就可以建立通信套接字在这个通信套接字上进行读写操作，此时不能再接收其他客户端连接请求，只能等待同当前连接的客户端的操作执行完成， 不过可以通过多线程来支持多个客户端的连接。

如果要让 **BIO 通信模型** 能够同时处理多个客户端请求，就必须使用多线程（主要原因是socket.accept()、socket.read()、socket.write() 涉及的三个主要函数都是同步阻塞的），也就是说它在接收到客户端连接请求之后为每个客户端创建一个新的线程进行链路处理，处理完成之后，通过输出流返回应答给客户端，线程销毁。这就是典型的 **一请求一应答通信模型** 。我们可以设想一下如果这个连接不做任何事情的话就会造成不必要的线程开销，不过可以通过 **线程池机制** 改善，线程池还可以让线程的创建和回收成本相对较低。

在 Java 虚拟机中，线程是宝贵的资源，线程的创建和销毁成本很高，除此之外，线程的切换成本也是很高的。尤其在 Linux 这样的操作系统中，线程本质上就是一个进程，创建和销毁线程都是重量级的系统函数。如果并发访问量增加会导致线程数急剧膨胀可能会导致线程堆栈溢出、创建新线程失败等问题，最终导致进程宕机或者僵死，不能对外提供服务。

在活动连接数不是特别高（小于单机1000）的情况下，这种模型是比较不错的，可以让每一个连接专注于自己的 I/O 并且编程模型简单，也不用过多考虑系统的过载、限流等问题。线程池本身就是一个天然的漏斗，可以缓冲一些系统处理不了的连接或请求。但是，当面对十万甚至百万级连接的时候，传统的 BIO 模型是无能为力的。因此，我们需要一种更高效的 I/O 处理模型来应对更高的并发量。

## 3. NIO (New I/O) ##

### 简介 ###

NIO是一种同步非阻塞的I/O模型，在Java 1.4 中引入了 NIO 框架，对应 java.nio 包，提供了 Channel , Selector，Buffer等抽象。

NIO中的N可以理解为Non-blocking，不单纯是New。它支持面向缓冲的，基于通道的I/O操作方法。 NIO提供了与传统BIO模型中的 Socket 和 ServerSocket 相对应的 SocketChannel 和 ServerSocketChannel 两种不同的套接字通道实现,两种通道都支持阻塞和非阻塞两种模式。阻塞模式使用就像传统中的支持一样，比较简单，但是性能和可靠性都不好；非阻塞模式正好与之相反。对于低负载、低并发的应用程序，可以使用同步阻塞I/O来提升开发速率和更好的维护性；对于高负载、高并发的（网络）应用，应使用 NIO 的非阻塞模式来开发。

### NIO的特性/NIO与IO区别 ###

#### 1）非阻塞IO ####

IO是阻塞的，NIO是不阻塞的。

Java NIO使我们可以进行非阻塞IO操作。比如说，单线程中从通道读取数据到buffer，同时可以继续做别的事情，当数据读取到buffer中后，线程再继续处理数据。写数据也是一样的。另外，非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。

Java IO的各种流是阻塞的。这意味着，当一个线程调用 read() 或 write() 时，该线程被阻塞，直到有一些数据被读取，或数据完全写入。该线程在此期间不能再干任何事情了。

#### 2)Buffer(缓冲区) ####

IO 面向流(Stream oriented)，而 NIO 面向缓冲区(Buffer oriented)。

Buffer是一个对象，它包含一些要写入或者要读出的数据。在NIO类库中加入Buffer对象，体现了新库与原I/O的一个重要区别。在面向流的I/O中·可以将数据直接写入或者将数据直接读到 Stream 对象中。虽然 Stream 中也有 Buffer 开头的扩展类，但只是流的包装类，还是从流读到缓冲区，而 NIO 却是直接读到 Buffer 中进行操作。

在NIO厍中，所有数据都是用缓冲区处理的。在读取数据时，它是直接读到缓冲区中的; 在写入数据时，写入到缓冲区中。任何时候访问NIO中的数据，都是通过缓冲区进行操作。

最常用的缓冲区是 ByteBuffer,一个 ByteBuffer 提供了一组功能用于操作 byte 数组。除了ByteBuffer,还有其他的一些缓冲区，事实上，每一种Java基本类型（除了Boolean类型）都对应有一种缓冲区。

#### 3)Channel (通道) ####

NIO 通过Channel（通道） 进行读写。

通道是双向的，可读也可写，而流的读写是单向的。无论读写，通道只能和Buffer交互。因为 Buffer，通道可以异步地读写。

#### 4)Selector (选择器) ####

NIO有选择器，而IO没有。

选择器用于使用单个线程处理多个通道。因此，它需要较少的线程来处理这些通道。线程之间的切换对于操作系统来说是昂贵的。 因此，为了提高系统效率选择器是有用的。

## 3. AIO (Asynchronous I/O) ##

AIO 也就是 NIO 2。在 Java 7 中引入了 NIO 的改进版 NIO 2,它是异步非阻塞的IO模型。

异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。


