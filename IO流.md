# 一、字符流

## 文件的写入：FileWriter

- 先学习一下字符流的特点。既然IO流是用于操作数据的，那么数据的最常见体现形式是：文件。
- 那么先以操作文件为主来演示。
- 需求:在硬盘上，创建一个文件并写入一些文字数据。
- 找到一个专门用于操作文件的Writer子类对象：FileWriter。  后缀名是父类名。 前缀名是该流对象的功能。

~~~java
import java.io.*;
class  FileWriterDemo
{
	public static void main(String[] args) throws IOException
	{
		//创建一个FileWriter对象。该对象一被初始化就必须要明确被操作的文件。
		//而且该文件会被创建到指定目录下。如果该目录下已有同名文件，将被覆盖。
		//其实该步就是在明确数据要存放的目的地。
		FileWriter fw = new FileWriter("demo.txt");

		//调用write方法，将字符串写入到流中。
		fw.write("abcde");

		//刷新流对象中的缓冲中的数据。
		//将数据刷到目的地中。
		//fw.flush();


		//关闭流资源，但是关闭之前会刷新一次内部的缓冲中的数据。
		//将数据刷到目的地中。
		//和flush区别：flush刷新后，流可以继续使用，close刷新后，会将流关闭。
		fw.close();
	}
}

~~~



## IO流异常处理方式

~~~java
import java.io.*;

class  FileWriterDemo2
{
	public static void main(String[] args) 
	{
		FileWriter fw = null;
		try
		{
			fw = new FileWriter("demo.txt");
			fw.write("abcdefg");

		}
		catch (IOException e)
		{
			System.out.println("catch:"+e.toString());
		}
		finally
		{
			try
			{
				if(fw!=null) //一定要判断该流对象是否为空，否则会出现空指针异常。
					fw.close();				
			}
			catch (IOException e)
			{
				System.out.println(e.toString());
			}
			
		}		

	}
}

~~~



## 文件的续写

- 演示对已有文件的数据续写

~~~java
import java.io.*;
class  FileWriterDemo3
{
	public static void main(String[] args) throws IOException
	{

		//传递一个true参数，代表不覆盖已有的文件,并在已有文件的末尾处进行数据续写。
		FileWriter fw = new FileWriter("demo.txt",true);

		fw.write("nihao\r\nxiexie");  // nihao后换行

		fw.close();
	}
}

~~~



## 文件的读取：FileReader 

### 1. 调用读取流对象的read方法

- 找到一个专门用于操作文件的Reader子类对象：FileReader

~~~java
import java.io.*;

class  FileReaderDemo
{
	public static void main(String[] args) throws IOException
	{
		//创建一个文件读取流对象，和指定名称的文件相关联。
		//要保证该文件是已经存在的，如果不存在，会发生异常FileNotFoundException
		FileReader fr = new FileReader("demo.txt");

		//调用读取流对象的read方法。
		//read():一次读一个字符，而且会自动往下读。
		
		int ch = 0;

		while((ch=fr.read())!=-1)
		{
			System.out.println((char)ch);
		}

		/*
		while(true)
		{
			int ch = fr.read();
			if(ch==-1)
				break;
			System.out.println("ch="+(char)ch);
		}
		*/
		fr.close();
	}
}

~~~

### 2. 通过字符数组进行读取。 

```java
  import java.io.*;

class FileReaderDemo2 
{
	public static void main(String[] args) throws IOException
	{
		FileReader fr = new FileReader("demo.txt");
		
		//定义一个字符数组。用于存储读到字符。
		//该read(char[])返回的是读到字符个数。
		char[] buf = new char[1024];

		int num = 0;
		while((num=fr.read(buf))!=-1)
		{
			System.out.println(new String(buf,0,num));
		}
		fr.close();
	}
}

```
## 拷贝文本文件

-  将C盘一个文本文件复制到D盘。

- 复制的原理：其实就是将C盘下的文件数据存储到D盘的一个文件中。


步骤：
1，在D盘创建一个文件。用于存储C盘文件中的数据。
2，定义读取流和C盘文件关联。
3，通过不断的读写完成数据存储。
4，关闭资源。

~~~java
import java.io.*;

class CopyText 
{
	public static void main(String[] args) throws IOException
	{
		copy_2();
	}

	public static void copy_2()
	{
		FileWriter fw = null;
		FileReader fr = null;
		try
		{
			fw = new FileWriter("SystemDemo_copy.txt");
			fr = new FileReader("SystemDemo.java");

			char[] buf = new char[1024];

			int len = 0;
			while((len=fr.read(buf))!=-1)
			{
				fw.write(buf,0,len);
			}
		}
		catch (IOException e)
		{
			throw new RuntimeException("读写失败");

		}
		finally
		{
			if(fr!=null)
				try
				{
					fr.close();
				}
				catch (IOException e)
				{
				}
			if(fw!=null)
				try
				{
					fw.close();
				}
				catch (IOException e)
				{
				}
		}
	}

	//从C盘读一个字符，就往D盘写一个字符。
	public static void copy_1()throws IOException
	{
		//创建目的地。
		FileWriter fw = new FileWriter("RuntimeDemo_copy.txt");

		//与已有文件关联。
		FileReader fr = new FileReader("RuntimeDemo.java");

		int ch = 0;

		while((ch=fr.read())!=-1)
		{
			fw.write(ch);
		}		
		fw.close();
		fr.close();
	}
}
~~~

# 二、字符流的缓冲区

## BufferedWriter

-  缓冲区是为了提高流的操作效率而出现的。所以在创建缓冲区之前，必须要先有流对象。
- 字符流写入缓冲区
- 该缓冲区中提供了一个跨平台的换行符：newLine();

~~~java
import java.io.*;

class  BufferedWriterDemo
{
	public static void main(String[] args) throws IOException
	{
		//创建一个字符写入流对象。
		FileWriter fw = new FileWriter("buf.txt");

		//为了提高字符写入流效率。加入了缓冲技术。
		//只要将需要被提高效率的流对象作为参数传递给缓冲区的构造函数即可。
		BufferedWriter bufw = new BufferedWriter(fw);

		for(int x=1; x<5; x++)
		{
			bufw.write("abcd"+x);
			bufw.newLine();
			bufw.flush();
		}

		//记住，只要用到缓冲区，就要记得刷新。
		//bufw.flush();

		//其实关闭缓冲区，就是在关闭缓冲区中的流对象。
		bufw.close();
	}
}

~~~

## BufferedReader

- 字符流读取缓冲区
- 该缓冲区提供了一个一次读一行的方法 readLine，方便于对文本数据的获取。当返回null时，表示读到文件末尾。
- readLine方法返回的时候只返回回车符之前的数据内容，并不返回回车符。

~~~Java
import java.io.*;

class  BufferedReaderDemo
{
	public static void main(String[] args) throws IOException
	{
		//创建一个读取流对象和文件相关联。
		FileReader fr = new FileReader("buf.txt");

		//为了提高效率。加入缓冲技术。将字符读取流对象作为参数传递给缓冲对象的构造函数。
		BufferedReader bufr = new BufferedReader(fr);		

		String line = null;

		while((line=bufr.readLine())!=null)
		{
			System.out.print(line);
		}

		bufr.close();
	}

}
~~~

## 缓冲区复制文本文件

- 通过缓冲区复制一个.java文件。

~~~java
import java.io.*;

class  CopyTextByBuf
{
	public static void main(String[] args) 
	{
		BufferedReader bufr = null;
		BufferedWriter bufw = null;

		try
		{
			bufr = new BufferedReader(new FileReader("BufferedWriterDemo.java"));
			bufw = new BufferedWriter(new FileWriter("bufWriter_Copy.txt"));

			String line = null;

			while((line=bufr.readLine())!=null)
			{
				bufw.write(line);
				bufw.newLine();
				bufw.flush();

			}
		}
		catch (IOException e)
		{
			throw new RuntimeException("读写失败");
		}
		finally
		{
			try
			{
				if(bufr!=null)
					bufr.close();
			}
			catch (IOException e)
			{
				throw new RuntimeException("读取关闭失败");
			}
			try
			{
				if(bufw!=null)
					bufw.close();
			}
			catch (IOException e)
			{
				throw new RuntimeException("写入关闭失败");
			}
		}
	}
}

~~~

## 自定义的MyBufferedReader

-  明白了BufferedReader类中特有方法readLine的原理后，可以自定义一个类中包含一个功能和readLine一致的方法。
- 用来模拟一下BufferedReader

~~~java
import java.io.*;
class MyBufferedReader 
{
	private /*File*/Reader r;
	MyBufferedReader(/*File*/Reader r)
	{
		this.r = r;
	}

	//可以一次读一行数据的方法。
	public String myReadLine()throws IOException
	{
		//定义一个临时容器。原BufferReader封装的是字符数组。
		//为了演示方便。定义一个StringBuilder容器。因为最终还是要将数据变成字符串。
		StringBuilder sb = new StringBuilder();
		int ch = 0;
		while((ch=r.read())!=-1)
		{
			if(ch=='\r')
				continue;
			if(ch=='\n')
				return sb.toString();
			else
				sb.append((char)ch);
		}
		if(sb.length()!=0)
			return sb.toString();
		return null;		
	}
	
    public void myclose()
    {
        r.close();
    }
}	

class  MyBufferedReaderDemo
{
	public static void main(String[] args) throws IOException
	{
		FileReader fr = new FileReader("buf.txt");

		MyBufferedReader myBuf = new MyBufferedReader(fr);

		String line = null;

		while((line=myBuf.myReadLine())!=null)
		{
			System.out.println(line);
		}
		myBuf.myClose();
	}
}

~~~

## 装饰设计模式

- 装饰设计模式：当想要对已有的对象进行功能增强时，可以定义类，将已有对象传入，基于已有的功能，并提供加强功能。那么自定义的该类称为装饰类。
- 装饰类通常会通过构造方法接收被装饰的对象，并基于被装饰的对象的功能，提供更强的功能。

~~~java
class Person
{
	public void chifan()
	{
		System.out.println("吃饭");
	}
}

class SuperPerson 
{
	private Person p ;
	SuperPerson(Person p)
	{
		this.p = p;
	}
	public void superChifan()
	{
		System.out.println("开胃酒");
		p.chifan();
		System.out.println("甜点");
		System.out.println("来一根");
	}
}

class  PersonDemo
{
	public static void main(String[] args) 
	{
		Person p = new Person();

		//p.chifan();

		SuperPerson sp = new SuperPerson(p);
		sp.superChifan();

	}
}

~~~

## 装饰与继承的区别

 MyReader//专门用于读取数据的类。
	|--MyTextReader
		|--MyBufferTextReader
	|--MyMediaReader
		|--MyBufferMediaReader
	|--MyDataReader
		|--MyBufferDataReader

~~~java
class MyBufferReader
{
	MyBufferReader(MyTextReader text)
	{}
	MyBufferReader(MyMediaReader media)
	{}
}
~~~

上面这个类扩展性很差。
找到其参数的共同类型，通过多态的形式，可以提高扩展性。

~~~java
class MyBufferReader extends MyReader
{
	private MyReader r;
	MyBufferReader(MyReader r)
	{}
}	
~~~


MyReader//专门用于读取数据的类。
	|--MyTextReader
	|--MyMediaReader
	|--MyDataReader
	|--MyBufferReader


以前是通过继承让每一个子类都具备缓冲功能。那么继承体系会复杂，并不利于扩展。

现在优化思想，单独描述一下缓冲内容。
将需要被缓冲的对象，传递进来。也就是，谁需要被缓冲，谁就作为参数传递给缓冲区。
这样继承体系就变得很简单，优化了体系结构。

装饰模式比继承要灵活，避免了继承体系臃肿，而且降低了类于类之间的关系。

装饰类因为增强已有对象，具备的功能和已有的是相同的，只不过提供了更强功能，
所以装饰类和被装饰类通常是都属于一个体系的。

- 使用装饰的自定义BufferedReader

~~~java
import java.io.*;
class MyBufferedReader extends Reader
{	
	private Reader r;
	MyBufferedReader(Reader r)
	{
		this.r = r;
	}

	//可以一次读一行数据的方法。
	public String myReadLine()throws IOException
	{
		//定义一个临时容器。原BufferReader封装的是字符数组。
		//为了演示方便。定义一个StringBuilder容器。因为最终还是要将数据变成字符串。
		StringBuilder sb = new StringBuilder();
		int ch = 0;
		while((ch=r.read())!=-1)
		{
			if(ch=='\r')
				continue;
			if(ch=='\n')
				return sb.toString();
			else
				sb.append((char)ch);
		}

		if(sb.length()!=0)
			return sb.toString();
		return null;		
	}

	/*
	覆盖Reader类中的抽象方法。
	*/
	public int read(char[] cbuf, int off, int len) throws IOException
	{
		return r.read(cbuf,off,len) ;
	}

	public void close()throws IOException
	{
		r.close();
	}
	public void myClose()throws IOException
	{
		r.close();
	}
}


class  MyBufferedReaderDemo
{
	public static void main(String[] args) throws IOException
	{
		FileReader fr = new FileReader("buf.txt");

		MyBufferedReader myBuf = new MyBufferedReader(fr);

		String line = null;

		while((line=myBuf.myReadLine())!=null)
		{
			System.out.println(line);
		}
		myBuf.myClose();
	}
}
~~~

## LineNumberReader

- setLineNumber():设置起始行号
- getLineNumber():获得行号

 ~~~java
import java.io.*;
class LineNumberReaderDemo 
{
	public static void main(String[] args)throws IOException 
	{
		FileReader fr = new FileReader("PersonDemo.java");

		LineNumberReader lnr = new LineNumberReader(fr);

		String line = null;
		lnr.setLineNumber(100);
		while((line=lnr.readLine())!=null)
		{
			System.out.println(lnr.getLineNumber()+":"+line);
		}
		lnr.close();
	}
}
 ~~~

##  自定义的MyLineNumberReader

~~~java
import java.io.*;

class MyLineNumberReader extends MyBufferedReader
{
	private int lineNumber;
	MyLineNumberReader(Reader r)
	{
		super(r);
	}

	public String myReadLine()throws IOException
	{
		lineNumber++;
		return super.myReadLine();
	}
	public void setLineNumber(int lineNumber)
	{
		this.lineNumber = lineNumber;
	}
	public int getLineNumber()
	{
		return lineNumber;
	}
}

/*
class MyLineNumberReader 
{
	private Reader r;
	private int lineNumber;
	MyLineNumberReader(Reader r)
	{
		this.r = r;
	}

	public String myReadLine()throws IOException
	{
		lineNumber++;
		StringBuilder sb = new StringBuilder();

		int ch = 0;

		while((ch=r.read())!=-1)
		{
			if(ch=='\r')
				continue;
			if(ch=='\n')
				return sb.toString();
			else
				sb.append((char)ch);
		}
		if(sb.length()!=0)
			return sb.toString();
		return null;
	}
	public void setLineNumber(int lineNumber)
	{
		this.lineNumber = lineNumber;
	}
	public int getLineNumber()
	{
		return lineNumber;
	}

	public void myClose()throws IOException
	{
		r.close();
	}
}
*/
class  MyLineNumberReaderDemo
{
	public static void main(String[] args) throws IOException
	{
		FileReader fr = new FileReader("copyTextByBuf.java");

		MyLineNumberReader mylnr = new MyLineNumberReader(fr);

		String line = null;
		mylnr.setLineNumber(100);
		while((line=mylnr.myReadLine())!=null)
		{
			System.out.println(mylnr.getLineNumber()+"::"+line);
		}

		mylnr.myClose();
	}
} 
~~~

# 三、字节流

## InputStream  &  OutputStream

- 如果要操作图片数据，这时就要用到字节流。
- 专门用于操作文件的InputStream子类对象：FileInputStream以及OutputStream字类对象：FileOutputStream。

~~~java
import java.io.*;
class  FileStream
{
	public static void main(String[] args) throws IOException
	{
		readFile_3();
	}

	public static void readFile_3()throws IOException
	{
		FileInputStream fis = new FileInputStream("fos.txt");
		
//		int num = fis.available();
		byte[] buf = new byte[fis.available()];//定义一个刚刚好的缓冲区，不用再循环了。数据过大慎用。

		fis.read(buf);

		System.out.println(new String(buf));

		fis.close();
	}

	public static void readFile_2()throws IOException
	{
		FileInputStream fis = new FileInputStream("fos.txt");

		byte[] buf = new byte[1024];
		int len = 0;
		while((len=fis.read(buf))!=-1)
		{
			System.out.println(new String(buf,0,len));
		}
		fis.close();		
	}

	public static void readFile_1()throws IOException
	{
		FileInputStream fis = new FileInputStream("fos.txt");

		int ch = 0;

		while((ch=fis.read())!=-1)
		{
			System.out.println((char)ch);
		}
		fis.close();
	}

	public static void writeFile()throws IOException
	{
		FileOutputStream fos = new FileOutputStream("fos.txt");		

		fos.write("abcde".getBytes());

		fos.close();		
	}
}
~~~

## 图片的复制

如何复制一张图片?

1，用字节读取流对象和图片关联。
2，用字节写入流对象创建一个图片文件，用于存储获取到的图片数据。
3，通过循环读写，完成数据的存储。
4，关闭资源。

~~~java
import java.io.*;
class  CopyPic
{
	public static void main(String[] args) 
	{
		FileOutputStream fos = null;
		FileInputStream fis = null;
		try
		{
			fos = new FileOutputStream("c:\\2.bmp");
			fis = new FileInputStream("c:\\1.bmp");

			byte[] buf = new byte[1024];

			int len = 0;

			while((len=fis.read(buf))!=-1)
			{
				fos.write(buf,0,len);
			}
		}
		catch (IOException e)
		{
			throw new RuntimeException("复制文件失败");
		}
		finally
		{
			try
			{
				if(fis!=null)
					fis.close();
			}
			catch (IOException e)
			{
				throw new RuntimeException("读取关闭失败");
			}
			try
			{
				if(fos!=null)
					fos.close();
			}
			catch (IOException e)
			{
				throw new RuntimeException("写入关闭失败");
			}
		}
	}
}
~~~

## 字节流的缓冲区

通过缓冲区实现mp3文件的复制：BufferedOutputStream   BufferedInputStream

~~~java
import java.io.*;
class  CopyMp3
{
	public static void main(String[] args) throws IOException
	{
		long start = System.currentTimeMillis();
		copy_2();
		long end = System.currentTimeMillis();

		System.out.println((end-start)+"毫秒");
	}

	public static void copy_2()throws IOException
	{
		MyBufferedInputStream bufis = new MyBufferedInputStream(new                                                                                            FileInputStream("c:\\9.mp3"));
		BufferedOutputStream bufos = new BufferedOutputStream(new FileOutputStream("c:\\3.mp3"));
		
		int by = 0;

		//System.out.println("第一个字节:"+bufis.myRead());

		while((by=bufis.myRead())!=-1)
		{
			bufos.write(by);
		}

		bufos.close();
		bufis.myClose();
	}

	//通过字节流的缓冲区完成复制。
	public static void copy_1()throws IOException
	{
		BufferedInputStream bufis = new BufferedInputStream(new FileInputStream("c:\\0.mp3"));
		BufferedOutputStream bufos = new BufferedOutputStream(new FileOutputStream("c:\\1.mp3"));
		
		int by = 0;

		while((by=bufis.read())!=-1)
		{
			bufos.write(by);
		}

		bufos.close();
		bufis.close();
		
	}
}
~~~

## 自定义的字节流缓冲区

~~~java
import java.io.*;

class MyBufferedInputStream
{
	private InputStream in;

	private byte[] buf = new byte[1024*4];
		
	private int pos = 0,count = 0;
	
	MyBufferedInputStream(InputStream in)
	{
		this.in = in;
	}

	//一次读一个字节，从缓冲区(字节数组)获取。
	public int myRead()throws IOException
	{
		//通过in对象读取硬盘上数据，并存储buf中。
		if(count==0)
		{
			count = in.read(buf);
			if(count<0)
				return -1;
			pos = 0;
			byte b = buf[pos];

			count--;
			pos++;
			return b&255;
		}
		else if(count>0)
		{
			byte b = buf[pos];

			count--;
			pos++;
			return b&0xff;
		}
		return -1;
	}
    
	public void myClose()throws IOException
	{
		in.close();
	}
}
~~~

说明：

byte: -1  --->  int : -1;
00000000 00000000 00000000 11111111  255

11111111 11111111 11111111 11111111

11111111  -->提升了一个int类型 那不还是-1吗？是-1是因为在8个1前面补的是1。
只要在前面补0，既可以保留原字节数据不变，又可以避免-1的出现。
怎么补0呢？

​    11111111    1111111    11111111  11111111                        

<u>& 00000000 00000000 00000000 11111111</u>

   00000000 00000000 00000000 11111111 

0000-0001
1111-1110
000000001
1111-1111  -1

结论：
字节流的读一个字节的read方法为什么返回值类型不是byte，而是int。
因为有可能会读到连续8个二进制1的情况，8个二进制1对应的十进制是-1，
那么就会出现数据还没有读完就结束的情况。因为我们判断读取结束是通过结尾标记-1来确定的。
所以，为了避免这种情况将读到的字节进行int类型的提升，
并在保留原字节数据的情况前面了补了24个0，变成了int类型的数值。
而在写入数据时，只写该int类型数据的最低8位。

---

## 读取键盘录入

- System.out: 对应的是标准输出设备，控制台。
- System.in: 对应的标准输入设备，键盘。

~~~java
/*
通过键盘录入数据。当录入一行数据后，就将该行数据进行打印。
如果录入的数据是over，那么停止录入。
*/
import java.io.*;
class  ReadIn
{
	public static void main(String[] args) throws IOException
	{
		InputStream in = System.in;
		StringBuilder sb = new StringBuilder();

		while(true)
		{
			int ch = in.read();
			if(ch=='\r')
				continue;
			if(ch=='\n')
			{
				String s = sb.toString();
				if("over".equals(s))
					break;
				System.out.println(s.toUpperCase());   //大写打印
				sb.delete(0,sb.length());  //清空缓存区
			}
			else
				sb.append((char)ch); 
		}
	}
}
~~~

# 四、读写转换流

## 读取转换流 

读取的字节流转换成字符流——InputStreamReader。

通过上面的键盘录入一行数据并打印其大写，发现其实就是读一行数据的原理，即readLine方法。
能不能直接使用readLine方法来完成键盘录入的一行数据的读取呢？
readLine方法是字符流BufferedReader类中的方法。
键盘录入的read方法是字节流InputStream的方法。
那么能不能将字节流转成字符流再使用字符流缓冲区的readLine方法呢？

~~~java
import java.io.*;

class  TransStreamDemo
{
	public static void main(String[] args) throws IOException
	{
		//获取键盘录入对象。
		InputStream in = System.in;

		//将字节流对象转成字符流对象，使用转换流：InputStreamReader
		InputStreamReader isr = new InputStreamReader(in);

		//为了提高效率，将字符流进行缓冲区技术高效操作。使用BufferedReader
         BufferedReader bufr = new BufferedReader(isr);
        
         String line = null;

		while((line=bufr.readLine())!=null)
		{
			if("over".equals(line))
				break;
             system.out.println(line.toUpperCase());
        }
        
        bufr.close();
    }
}
~~~

## 写入转换流

写入的字节流转换成字符流——OutputStreamWriter。

- 源：键盘录入
- 目的：控制台

~~~java
import java.io.*;

class  TransStreamDemo
{
	public static void main(String[] args) throws IOException
	{
		//键盘录入的最常见写法。
		BufferedReader bufr = new BufferedReader(new InputStreamReader(System.in));
        
        /*
        源：文件  目的：控制台
        将第8行代码改为：
        BufferedReader bufr = 
                     new BufferedReader(new InputStreamReader(new FileInputStream("*.java")));
		*/
        
//		OutputStream out = System.out;
//		OutputStreamWriter osw = new OutputStreamWriter(out);
//		BufferedWriter bufw = new BufferedWriter(osw);
		BufferedWriter bufw = new BufferedWriter(new OutputStreamWriter(System.out));
        
        /*
        源：键盘录入   目的：文件
        13行代码改为：
        BufferedWriter bufw =
                      new BufferedWriter(new OutputStreamWriter(new FileOutputStream("*.txt")));
        */

		String line = null;

		while((line=bufr.readLine())!=null)
		{
			if("over".equals(line))
				break;
			bufw.write(line.toUpperCase());
			bufw.newLine();
			bufw.flush();
		}
		bufr.close();
	}
}

~~~

## 流的操作规律

流操作的一个难点就是流对象有很多，不知道该用哪一个。
可以通过三个明确来确定流对象。
1.明确源和目的。
    源：输入流。InputStream  Reader
    目的：输出流。OutputStream  Writer
2.明确操作的数据是否是纯文本。
     Y:字符流   N:字节流
3.当体系明确后，再明确要使用哪个具体的对象。
   	通过设备来进行区分：
              源设备：内存、硬盘、键盘
	      目的设备：内存、硬盘、控制台

---

具体操作流程：

1.将一个文本文件中数据存储到另一个文件中，即复制文件。
   源： 因为是源，所以使用读取流。InputStream or Reader 
           确定是不是操作文本文件。是，故选择Reader。这样体系就明确了。
           接下来明确要使用该体系中的哪个对象。
           明确设备：硬盘，即上一个文件。
           Reader体系中可以操作文件的对象：FileReader
           明确是否需要提高效率：是，则加入Reader体系中缓冲区 BufferedReader。

~~~java
FileReader fr = new FileReader("a.txt");
BufferedReader bufr = new BufferedReader(fr);
~~~

 目的：选择写入流，OutputStream or  Writer
            确定是不是操作文本文件。是，故选择Writer。
            明确设备：硬盘，即另一个文件。
            Writer体系中可以操作文件的对象：FileWriter。
            是否需要提高效率：是，加入Writer体系中缓冲区 BufferedWriter。

~~~java
FileWriter fw = new FileWriter("b.txt");
BufferedWriter bufw = new BufferedWriter(fw);
~~~

---

2. 将键盘录入的数据保存到一个文件中。

源：InputStream or Reader
       是不是纯文本？是，Reader
       设备：键盘。对应的对象是System.in.
       不是选择Reader吗？System.in对应的不是字节流吗？
       为了操作键盘的文本数据方便，转成字符流按照字符串操作是最方便的。
       所以既然明确了Reader，那么就将System.in转换成Reader。
       用了Reader体系中转换流，InputStreamReader

​       InputStreamReader isr = new InputStreamReader(System.in);

​       需要提高效率吗？需要，BufferedReader
       BufferedReader bufr = new BufferedReader(isr);

目的：OutputStream or Writer
          是否是存文本？是，Writer。
          设备：硬盘，即一个文件。使用 FileWriter。

​          FileWriter fw = new FileWriter("c.txt");

​          需要提高效率吗？需要，BufferedWriter
          BufferedWriter bufw = new BufferedWriter(fw);

---

扩展：把录入的数据按照指定的编码表（utf-8），将数据存到文件中。
	
目的：OutputStream  Writer
          是否是存文本？是！Writer。
          设备：硬盘，即一个文件。使用 FileWriter。
          但是FileWriter是使用的默认编码表：GBK.

​          在存储时，需要加入指定编码表utf-8。而指定的编码表只有转换流可以指定。
          所以要使用的对象是OutputStreamWriter。
          该转换流对象要接收一个字节输出流。
          可以操作文件的字节输出流：FileOutputStream

OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream("d.txt"),"UTF-8");

需要高效吗？需要。
BufferedWriter bufw = new BufferedWriter(osw);

所以，记住：转换流是字符和字节之间的桥梁。通常，涉及到字符编码转换时，需要用到转换流。

---

## 改变标准输入输出设备

- 改变标准输入设备：setIn(InputStream in)
- 改变标准输出设备：setOut(PrintStream out)

~~~java
class  TransStreamDemo2
{
	public static void main(String[] args) throws IOException
	{
        //从文本输入到文本输出，就是复制。
		System.setIn(new FileInputStream("PersonDemo.java"));

		System.setOut(new PrintStream("zzz.txt"));

		//键盘的最常见写法。
		BufferedReader bufr = 
				new BufferedReader(new InputStreamReader(System.in));
		
		BufferedWriter bufw = new BufferedWriter(new OutputStreamWriter(System.out));

		String line = null;

		while((line=bufr.readLine())!=null)
		{
			if("over".equals(line))
				break;
			bufw.write(line.toUpperCase());
			bufw.newLine();
			bufw.flush();
		}
		bufr.close();
	}
}
~~~

## 异常的日志信息

​        ——将异常的日志信息打印到文件中。

~~~java
import java.io.*;
import java.util.*;
import java.text.*;
class  ExceptionInfo
{
	public static void main(String[] args)throws IOException 
	{
		try
		{
			int[] arr = new int[2];
			System.out.println(arr[3]);  //指针超出异常
		}
		catch (Exception e)
		{			
			try
			{
				Date d = new Date();
				SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
				String s = sdf.format(d);

				PrintStream ps = new PrintStream("exeception.log");
				ps.println(s);
				System.setOut(ps);

				
			}
			catch (IOException ex)
			{
				throw new RuntimeException("日志文件创建失败");
			}
			e.printStackTrace(System.out);
		}
	}
}
//log4j
~~~

## 系统信息

​         ——打印系统信息到文件中

~~~java
import java.util.*;
import java.io.*;
class  SystemInfo
{
	public static void main(String[] args) throws IOException
	{
		Properties prop = System.getProperties();

		//System.out.println(prop);
		prop.list(new PrintStream("sysinfo.txt"));
	}
}
~~~

# 五、File

## File概述

 File类：

- 用来将文件或者文件夹封装成对象
- 方便对文件与文件夹的属性信息进行操作 
- File对象可以作为参数传递给流的构造函数

~~~Java
//创建File对象
class FileDemo
{
    public static void main(String[] args)
    {
        consMethod();
    }
	public static void consMethod()
	{
		//将a.txt封装成file对象。可以将已有的和未出现的文件或者文件夹封装成对象。
		File f1 = new File("a.txt");  //相对路径

		File f2 = new File("c:\\abc","b.txt");   //绝对路径

         File d = new File("c:\\abc");
		File f3 = new File(d,"c.txt");
 
		sop("f1:"+f1);
		sop("f2:"+f2);
		sop("f3:"+f3);

		File f4 = new                                                                                                 File("c:"+File.separator+"abc"+File.separator+"zzz"+File.separator+"a.txt");
	}        //在不同操作系统下路径都适用

	public static void sop(Object obj) 
	{
		System.out.println(obj);
	}
}
~~~

## File对象功能

### 1.File类常见方法

  1，创建。
        boolean createNewFile():在指定位置创建文件，如果该文件已经存在，则不创建，返回false。
		                                和输出流不一样，输出流对象一建立就会创建文件。如果文件已经存在，则会覆盖。
       boolean mkdir():创建文件夹。
       boolean mkdirs():创建多级文件夹。

  2，删除。
	boolean delete()：删除失败返回false。如果文件正在被使用，则删除不了返回false。
	void deleteOnExit()：在程序退出时删除指定文件。

  3，判断。
	boolean exists() :文件是否存在。
	boolean isFile():判断是否是文件
	boolean isDirectory()：判断是否是目录
	boolean isHidden()：判断是否是隐藏文件
	boolean isAbsolute()：判断是否是绝对路径名

  4，获取信息。
	String getName():获取文件或目录名
	String getPath():获取路径
	String getParent():获取父目录

​        getAbsolutePath() : 获取绝对路径
	long lastModified() ：获取最后一次修改的时间
	long length() ：获取文件的大小

~~~java
class FileDemo 
{
	public static void main(String[] args) throws IOException
	{
		method_5();
	}

	public static void method_5()
	{
		File f1 = new File("c:\\Test.java");
		File f2 = new File("d:\\hahah.java");

		sop("rename:"+f2.renameTo(f1));   //f2重命名为f1
	}

	public static void method_4()
	{
		File f = new File("file.txt");

		sop("path:"+f.getPath());
		sop("abspath:"+f.getAbsolutePath());
		sop("parent:"+f.getParent());// 该方法返回的是绝对路径中的父目录。如果获取的是相对路径，返回null。
									//如果相对路径中有上一层目录那么该目录就是返回结果。
	}
	
	public static void method_3()throws IOException
	{
		File f = new File("d:\\java1223\\day20\\file2.txt");

		//f.createNewFile();

		//f.mkdir();

		//记住在判断文件对象是否是文件或者目录时，必须要先判断该文件对象封装的内容是否存在。
		//通过exists判断。
		sop("dir:"+f.isDirectory());
		sop("file:"+f.isFile());

		sop(f.isAbsolute());
	}

	public static void method_2()
	{
		File f = new File("file.txt");

		//sop("exists:"+f.exists());       //判断文件是否存在

		//sop("execute:"+f.canExecute());  //判断文件是否能执行

		//创建文件夹
		File dir = new File("abc\\kkk\\a\\a\\dd\\ee\\qq\\aaa");

		sop("mkdir:"+dir.mkdirs());
	}
	
	public static void method_1()throws IOException
	{
		File f = new File("file.txt");
         //sop("create:"+f.createNewFile());
		//sop("delete:"+f.delete());		
	}
    
    public static void sop(Object obj)
	{
		System.out.println(obj);
	}
~~~

### 2.文件列表

​     static File listRoots()：列出可用的文件系统根

​     String[] list()：返回路径中的文件和目录

​     String[] list(FilenameFilter filter)：返回路径中满足指定过滤器的文件和目录

​     File[] listFiles()：返回路径中的文件

~~~Java
import java.io.*;

class  FileDemo2
{
	public static void main(String[] args) 
	{
        //listDemo()；
        //listRootsDemo()；
        
		File dir = new File("c:\\");
		File[] files = dir.listFiles();

		for(File f : files)
		{
			System.out.println(f.getName()+"::"+f.length());
		}		
	}

	public static void listDemo_2()
	{
		File dir = new File("d:\\java1223\\day18");

		String[] arr = dir.list(new FilenameFilter()
		{
			public boolean accept(File dir,String name)
			{
//				System.out.println("dir:"+dir+"....name::"+name);
				/*
				if(name.endsWith(".bmp"))
					return true;
				else
				    return false;
				*/
				return name.endsWith(".bmp");
			}
		});

		System.out.println("len:"+arr.length);
		for(String name : arr)
		{
			System.out.println(name);
		}
	}

	public static void listDemo()
	{
		File f = new File("c:\\abc.txt");

		String[] names = f.list();//调用list方法的file对象必须是封装了一个目录。该目录还必须存在。
		for(String name : names)
		{
			System.out.println(name);
		}
	}
	public static void listRootsDemo()
	{
		File[] files = File.listRoots();

		for(File f : files)
		{
			System.out.println(f);
		}
	}
}
~~~

## 列出目录下所有内容—递归

列出指定目录下文件或者文件夹，包含子目录中的内容，也就是列出指定目录下所有内容。

因为目录中还有目录，只要使用同一个列出目录功能的函数完成即可。
在列出过程中出现的还是目录的话，还可以再次调用本功能，也就是函数自身调用自身。
这种表现形式，或者编程手法，称为递归。

递归要注意：
1，限定条件。
2，要注意递归的次数，尽量避免内存溢出

~~~java
import java.io.*;

class FileDemo3 
{
	public static void main(String[] args) 
	{
		File dir = new File("d:\\testdir");
		//showDir(dir,0);

		//toBin(6);
		//int n = getSum(8000);
		//System.out.println("n="+n);

		System.out.println(dir.delete());
	}
    
    //为目录添加层级结构
	public static String getLevel(int level)
	{
		StringBuilder sb = new StringBuilder();
		sb.append("|--");
		for(int x=0; x<level; x++)
		{
			//sb.append("|--");
			sb.insert(0,"|  ");

		}
		return sb.toString();
	}
    
	public static void showDir(File dir,int level)
	{		
		System.out.println(getLevel(level)+dir.getName());

		level++;
		File[] files = dir.listFiles();
		for(int x=0; x<files.length; x++)
		{
			if(files[x].isDirectory())
				showDir(files[x],level);
			else
				System.out.println(getLevel(level)+files[x]);
		}
	}

    /*
    //递归的两个实例
	public static int getSum(int n)
	{
		if(n==1)
			return 1;
		return n+getSum(n-1);
	}

	public static void toBin(int num)
	{
		if(num>0)
		{
			toBin(num/2);
			System.out.println(num%2);
		}
	}
*/
	public static void method()
	{
		method();
	}
}
~~~

## 删除带内容的目录

删除原理：
在windows中，删除目录从里面往外删除的。
既然是从里往外删除，就需要用到递归。

~~~java
import java.io.*;
class  RemoveDir
{
	public static void main(String[] args) 
	{
		
		File dir = new File("d:\\testdir");
		removeDir(dir);
	}

	public static void removeDir(File dir)
	{
		File[] files = dir.listFiles();
		
		for(int x=0; x<files.length; x++)
		{
			if(files[x].isDirectory())
				removeDir(files[x]);
			else
				System.out.println(files[x].toString()+":-file-:"+files[x].delete());
		}

		System.out.println(dir+"::dir::"+dir.delete());
	}
}
~~~

## 创建java文件列表

将一个指定目录下的java文件的绝对路径，存储到一个文本文件中，建立一个java文件列表文件。

思路：
1，对指定的目录进行递归。
2，获取递归过程所有的java文件的路径。
3，将这些路径存储到集合中。
4，将集合中的数据写入到一个文件中。

~~~java
import java.io.*;
import java.util.*;
class  JavaFileList
{
	public static void main(String[] args) throws IOException
	{
		
		File dir = new File("d:\\java1223");

		List<File> list = new ArrayList<File>();

		fileToList(dir,list);

		//System.out.println(list.size());

		File file = new File(dir,"javalist.txt");
		writeToFile(list,file.toString());		
	}
    
	public static void fileToList(File dir,List<File> list)
	{
		File[] files = dir.listFiles();

		for(File file : files)
		{
			if(file.isDirectory())
				fileToList(file,list);
			else
			{
				if(file.getName().endsWith(".java"))
					list.add(file);
			}
		}
	}

	public static void writeToFile(List<File> list,String javaListFile)throws IOException
	{
		BufferedWriter bufw =  null;
		try
		{
			bufw = new BufferedWriter(new FileWriter(javaListFile));
			
			for(File f : list)
			{
				String path = f.getAbsolutePath();
				bufw.write(path);
				bufw.newLine();
				bufw.flush();
			}
		}
		catch (IOException e)
		{
			throw e;
		}
		finally
		{
			try
			{
				if(bufw!=null)
					bufw.close();
			}
			catch (IOException e)
			{
				throw e;
			}
		}
	}
}
~~~

# 六、Properties

## Properties基础

- Properties是hashtable的子类，也就是说它具备map集合的特点，而且它里面存储的键值对都是字符串。
- 它是集合中和IO技术相结合的集合容器。

- 该对象的特点：可以用于键值对形式的配置文件。

- 在加载数据时，需要数据有固定格式：键=值。

~~~java
import java.io.*;
import java.util.*;

class PropertiesDemo 
{
	public static void main(String[] args) throws IOException
	{
        //setAndGet()
		//method_1();
		loadDemo();
	}

	public static void loadDemo()throws IOException
	{
		Properties prop = new Properties();
		FileInputStream fis = new FileInputStream("info.txt");

		//将流中的数据加载进集合。
		prop.load(fis);

		prop.setProperty("wangwu","39");  //修改数据

		FileOutputStream fos = new FileOutputStream("info.txt");

		prop.store(fos,"haha");    //储存修改后数据

	//	System.out.println(prop);
		prop.list(System.out);

		fos.close();
		fis.close();
	}

	//演示，如何将流中的数据存储到集合中。
	//想要将info.txt中键值数据存到集合中进行操作。
	/*
		1，用一个流和info.txt文件关联。
		2，读取一行数据，将该行数据用"="进行切割。
		3，等号左边作为键，右边作为值，存入到Properties集合中。
	*/
	public static void method_1()throws IOException
	{
		BufferedReader bufr = new BufferedReader(new FileReader("info.txt"));

		String line = null;
		Properties prop = new Properties();

		while((line=bufr.readLine())!=null)
		{
			String[] arr = line.split("=");
			///System.out.println(arr[0]+"...."+arr[1]);
			prop.setProperty(arr[0],arr[1]);
		}
		bufr.close();

		System.out.println(prop);
	}

//	设置和获取元素。
	public static void setAndGet()
	{
		Properties prop = new Properties();

		prop.setProperty("zhangsan","30");
		prop.setProperty("lisi","39");

//		System.out.println(prop);
		String value = prop.getProperty("lisi");
		//System.out.println(value);
			
		prop.setProperty("lisi",89+"");

		Set<String> names = prop.stringPropertyNames();
		for(String s : names)
		{
			System.out.println(s+":"+prop.getProperty(s));
		}
	}	
}
~~~

## Properties的应用

要求：记录应用程序运行次数。如果使用次数已到，那么给出注册提示。

​          很容易想到的是:计数器。
          该计数器定义在程序中，随着程序的运行而在内存中存在，并进行自增。
          可是随着该应用程序的退出，该计数器也在内存中消失了。
          下一次再启动该程序，又重新开始从0计数。这样不是我们想要的。

​         程序即使结束，该计数器的值也存在。
         下次程序启动会先加载该计数器的值并加1后再重新存储起来。
         所以要建立一个配置文件，用于记录该软件的使用次数。

​         该配置文件使用键值对的形式。这样便于阅读数据，并操作数据。

​         键值对数据属于map集合。数据是以文件形式存储，使用io技术。
         那么map+io -->properties.

​         配置文件可以实现应用程序数据的共享。

~~~java
import java.io.*;
import java.util.*;
class  RunCount
{
	public static void main(String[] args) throws IOException
	{
		Properties prop = new Properties();

		File file = new File("count.ini");
		if(!file.exists())
			file.createNewFile();
		
		FileInputStream fis = new FileInputStream(file);

		prop.load(fis);
		
		int count = 0;
		String value = prop.getProperty("time");
		
		if(value!=null)
		{
			count = Integer.parseInt(value);
			if(count>=5)
			{
				System.out.println("您好，使用次数已到，拿钱!");
				return ;
			}
		}

		count++;

		prop.setProperty("time",count+"");

		FileOutputStream fos = new FileOutputStream(file);

		prop.store(fos,"");

		fos.close();
		fis.close();		
	}
}
~~~

# 七、IO的其他类

## 打印流：PrintStream  PrintWriter

打印流：该流提供了打印方法，可以将各种数据类型的数据都原样打印。

字节打印流：PrintStream
构造函数可以接收的参数类型：
1，file对象。File
2，字符串路径。String
3，字节输出流。OutputStream

字符打印流：PrintWriter
构造函数可以接收的参数类型：
1，file对象。File
2，字符串路径。String
3，字节输出流。OutputStream
4，字符输出流。Writer。

~~~java
import java.io.*;

class  PrintStreamDemo
{
	public static void main(String[] args) throws IOException
	{
		BufferedReader bufr = 
			new BufferedReader(new InputStreamReader(System.in));

	   // PrintWriter out = new PrintWriter(System.out,true);  //打印在控制台，ture表示自动更新
        PrintWriter out = new PrintWriter(new FileWriter("a.txt"),true);//打印在硬盘文件

		String line = null;

		while((line=bufr.readLine())!=null)
		{
			if("over".equals(line))
				break;
			out.println(line.toUpperCase());
			//out.flush();
		}

		out.close();
		bufr.close();

	}	
}
~~~

## 序列流：SequenceInputStream

- ​ 序列流可以对多个流进行合并。


~~~Java
import java.io.*;
import java.util.*;
class SequenceDemo 
{
	public static void main(String[] args) throws IOException
	{
		Vector<FileInputStream> v = new Vector<FileInputStream>();
		
		v.add(new FileInputStream("c:\\1.txt"));
		v.add(new FileInputStream("c:\\2.txt"));
		v.add(new FileInputStream("c:\\3.txt"));

		Enumeration<FileInputStream> en = v.elements();

		SequenceInputStream sis = new SequenceInputStream(en);

		FileOutputStream fos = new FileOutputStream("c:\\4.txt");

		byte[] buf = new byte[1024];

		int len =0;
		while((len=sis.read(buf))!=-1)
		{
			fos.write(buf,0,len);
		}

		fos.close();
		sis.close();
	}
}
~~~

## 切割文件

 		——将文件切割然后再合并

~~~Java
import java.io.*;
import java.util.*;

class SplitFile 
{
	public static void main(String[] args) throws IOException
	{
		//splitFile();
		merge();
	}


	public static void merge()throws IOException   //合并
	{
		ArrayList<FileInputStream> al = new ArrayList<FileInputStream>();

		for(int x=1; x<=3; x++)
		{
			al.add(new FileInputStream("c:\\splitfiles\\"+x+".part"));
		}

		final Iterator<FileInputStream> it = al.iterator();

		Enumeration<FileInputStream> en = new Enumeration<FileInputStream>()
		{
			public boolean hasMoreElements()
			{
				return it.hasNext();
			}
			public FileInputStream nextElement()
			{
				return it.next();
			}
		};

		SequenceInputStream sis = new SequenceInputStream(en);

		FileOutputStream fos = new FileOutputStream("c:\\splitfiles\\0.bmp");

		byte[] buf = new byte[1024];

		int len = 0;

		while((len=sis.read(buf))!=-1)
		{
			fos.write(buf,0,len);
		}

		fos.close();
		sis.close();
	}

	public static void splitFile()throws IOException   //切割
	{
		FileInputStream fis =  new FileInputStream("c:\\1.bmp");

		FileOutputStream fos = null;

		byte[] buf = new byte[1024*1024];

		int len = 0;
		int count = 1;
		while((len=fis.read(buf))!=-1)
		{
			fos = new FileOutputStream("c:\\splitfiles\\"+(count++)+".part");
			fos.write(buf,0,len);
			fos.close();
		}		
		fis.close();		
	}
}
~~~

## 对象的序列化

-  ObjectInputStream
- ObjectOutputStream
- 被操作的对象需要实现Serializable(标记接口)

~~~java
//建立一个person对象，并对其序列化

import java.io.*;
class Person implements Serializable
{	
	public static final long serialVersionUID = 42L;

	private String name;
	transient int age;
	static String country = "cn";
	Person(String name,int age,String country)
	{
		this.name = name;
		this.age = age;
		this.country = country;
	}
	public String toString()
	{
		return name+":"+age+":"+country;
	}
}
~~~

~~~java
//实现对对象的存取
import java.io.*;

class ObjectStreamDemo 
{
	public static void main(String[] args) throws Exception
	{
		//writeObj();
		readObj();
	}
	public static void readObj()throws Exception
	{
		ObjectInputStream ois = new ObjectInputStream(new FileInputStream("obj.txt"));

		Person p = (Person)ois.readObject();

		System.out.println(p);
		ois.close();
	}

	public static void writeObj()throws IOException
	{
		ObjectOutputStream oos = 
			new ObjectOutputStream(new FileOutputStream("obj.txt"));

		oos.writeObject(new Person("lisi0",399,"kr"));

		oos.close();
	}
}

~~~

## 管道流

- PipedInputStream：输入管道流
- PipedOutputStream：输出管道流
- 输入输出可以通过管道流直接进行连接，结合线程使用

~~~java
import java.io.*;

class Read implements Runnable
{
	private PipedInputStream in;
	Read(PipedInputStream in)
	{
		this.in = in;
	}
	public void run()
	{
		try
		{
			byte[] buf = new byte[1024];

			System.out.println("读取前。。没有数据阻塞");
			int len = in.read(buf);
			System.out.println("读到数据。。阻塞结束");

			String s= new String(buf,0,len);

			System.out.println(s);

			in.close();
		}
		catch (IOException e)
		{
			throw new RuntimeException("管道读取流失败");
		}
	}
}

class Write implements Runnable
{
	private PipedOutputStream out;
	Write(PipedOutputStream out)
	{
		this.out = out;
	}
	public void run()
	{
		try
		{
			System.out.println("开始写入数据，等待6秒后。");
			Thread.sleep(6000);
			out.write("piped lai la".getBytes());
			out.close();
		}
		catch (Exception e)
		{
			throw new RuntimeException("管道输出流失败");
		}
	}
}

class  PipedStreamDemo
{
	public static void main(String[] args) throws IOException
	{

		PipedInputStream in = new PipedInputStream();
		PipedOutputStream out = new PipedOutputStream();
		in.connect(out);

		Read r = new Read(in);
		Write w = new Write(out);
		new Thread(r).start();
		new Thread(w).start();
	}
}
~~~

## 随机访问文件

 RandomAccessFile：该类不算是IO体系中的子类，而是直接继承自Object。

它是IO包中成员，因为它具备读和写功能。
内部封装了一个数组，而且通过指针对数组的元素进行操作。
可以通过getFilePointer获取指针位置，同时可以通过seek改变指针的位置。

其完成读写的原理就是内部封装了字节输入流和输出流。

通过构造函数可以看出，该类只能操作文件。
而且操作文件还有模式：只读r，读写rw等。

如果模式为只读 r，不会创建文件，会去读取一个已存在文件。如果该文件不存在，则会出现异常。
如果模式rw，操作的文件不存在，会自动创建。如果存在则不会覆盖。

~~~java
class RandomAccessFileDemo 
{
	public static void main(String[] args) throws IOException
	{
		//writeFile_2();
		//readFile();

		//System.out.println(Integer.toBinaryString(258));
	}

	public static void readFile()throws IOException
	{
		RandomAccessFile raf = new RandomAccessFile("ran.txt","r");
		
		//调整对象中指针。
		//raf.seek(8*1);

		//跳过指定的字节数
		raf.skipBytes(8);

		byte[] buf = new byte[4];

		raf.read(buf);

		String name = new String(buf);

		int age = raf.readInt();

		System.out.println("name="+name);
		System.out.println("age="+age);

		raf.close();
	}

	public static void writeFile_2()throws IOException
	{
		RandomAccessFile raf = new RandomAccessFile("ran.txt","rw");
		raf.seek(8*0);
		raf.write("周期".getBytes());
		raf.writeInt(103);

		raf.close();
	}

	public static void writeFile()throws IOException
	{
		RandomAccessFile raf = new RandomAccessFile("ran.txt","rw");

		raf.write("李四".getBytes());
		raf.writeInt(97);
		raf.write("王五".getBytes());
		raf.writeInt(99);

		raf.close();
	}
}
~~~

## 数据流

- 可以用于操作基本数据类型的数据的流对象：DataInputStream & DataOutputStream

~~~java
import java.io.*;
class DataStreamDemo 
{
	public static void main(String[] args) throws IOException
	{
		//writeData();
		//readData();

		//writeUTFDemo();

//		OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream("gbk.txt"),"gbk");
//
//		osw.write("你好");
//		osw.close();

//		readUTFDemo();

	}
	public static void readUTFDemo()throws IOException
	{
		DataInputStream dis = new DataInputStream(new FileInputStream("utf.txt"));

		String s = dis.readUTF();

		System.out.println(s);
		dis.close();
	}

	public static void writeUTFDemo()throws IOException
	{
		DataOutputStream dos = new DataOutputStream(new FileOutputStream("utfdate.txt"));

		dos.writeUTF("你好");

		dos.close();
	}

	public static void readData()throws IOException
	{
		DataInputStream dis = new DataInputStream(new FileInputStream("data.txt"));

		int num = dis.readInt();
		boolean b = dis.readBoolean();
		double d = dis.readDouble();

		System.out.println("num="+num);
		System.out.println("b="+b);
		System.out.println("d="+d);

		dis.close();
	}
	public static void writeData()throws IOException
	{
		DataOutputStream dos = new DataOutputStream(new FileOutputStream("data.txt"));

		dos.writeInt(234);
		dos.writeBoolean(true);
		dos.writeDouble(9887.543);

		dos.close();

		ObjectOutputStream oos = null;
		oos.writeObject(new O());		
	}
}
~~~

## 字节数组流

用于操作字节数组的流对象。

ByteArrayInputStream ：在构造的时候，需要接收数据源，而且数据源是一个字节数组。
ByteArrayOutputStream： 在构造的时候，不用定义数据目的，因为该对象中已经内部封装了可变长度的字节数组。
这就是数据目的地。

因为这两个流对象都操作的数组，并没有使用系统资源。所以，不用进行close关闭。

在流操作规律讲解时：

源设备：
	键盘 System.in，硬盘 FileStream，内存 ArrayStream。
目的设备：
	控制台 System.out，硬盘FileStream，内存 ArrayStream。

用流的读写思想来操作数据。

~~~java
import java.io.*;
class ByteArrayStream 
{
	public static void main(String[] args) 
	{
		//数据源。
		ByteArrayInputStream bis = new ByteArrayInputStream("ABCDEFD".getBytes());

		//数据目的
		ByteArrayOutputStream bos = new ByteArrayOutputStream();

		int by = 0;

		while((by=bis.read())!=-1)
		{
			bos.write(by);
		}

		System.out.println(bos.size());
		System.out.println(bos.toString());

	//	bos.writeTo(new FileOutputStream("a.txt"));
	}
}
~~~

# 八、编码

## 转换流的字符编码

~~~Java
import java.io.*;

class EncodeStream 
{
	public static void main(String[] args) throws IOException 
	{
			//writeText();
			readText();
	}

	public static void readText()throws IOException 
	{
		InputStreamReader isr = new InputStreamReader(new FileInputStream("utf.txt"),"gbk");

		char[] buf = new char[10];
		int len = isr.read(buf);

		String str = new String(buf,0,len);

		System.out.println(str);

		isr.close();
	}
	public static void writeText()throws IOException 
	{
		OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream("utf.txt"),"UTF-8");

		osw.write("你好");

		osw.close();
	}
}
~~~

## 字符的编解码

编码：字符串变成字节数组。
           String-->byte[]：str.getBytes(charsetName);

解码：字节数组变成字符串。
           byte[] -->String: new String(byte[],charsetName);

~~~java
import java.util.*;
class  EncodeDemo
{
	public static void main(String[] args)throws Exception 
	{
		String s = "哈哈";

		byte[] b1 = s.getBytes("GBK");

		System.out.println(Arrays.toString(b1));
		String s1 = new String(b1,"iso8859-1");
		System.out.println("s1="+s1);

		//对s1进行iso8859-1编码。
		byte[] b2 = s1.getBytes("iso8859-1");
		System.out.println(Arrays.toString(b2));

		String s2 = new String(b2,"gbk");

		System.out.println("s2="+s2);
    }
}
~~~

~~~java
class EncodeDemo2 
{
	public static void main(String[] args) throws Exception
	{
		String s = "联通";

		byte[] by = s.getBytes("gbk");

		for(byte b : by)
		{
			System.out.println(Integer.toBinaryString(b&255));
		}

	}
}
~~~

# 九、应用

有五个学生，每个学生有3门课的成绩，从键盘输入以上数据（包括姓名，三门课成绩）。
输入的格式：如：zhagnsan，30，40，60，计算出总成绩，
并把学生的信息和计算出的总分数按高低顺序存放在磁盘文件"stud.txt"中。

1，描述学生对象。
2，定义一个可操作学生对象的工具类。

思想：
1，通过获取键盘录入一行数据，并将该行中的信息取出封装成学生对象。
2，因为学生有很多，那么就需要存储，使用到集合。
     因为要对学生的总分排序，所以可以使用TreeSet。
3，将集合的信息写入到一个文件中。

~~~Java
import java.io.*;
import java.util.*;

class Student implements Comparable<Student>
{
	private String name;
	private int ma,cn,en;
	private int sum;

	Student(String name,int ma,int cn,int en)
	{
		this.name = name;
		this.ma = ma;
		this.cn = cn;
		this.en = en;
		sum = ma + cn + en;
	}


	public int compareTo(Student s)
	{
		int num = new Integer(this.sum).compareTo(new Integer(s.sum));
		if(num==0)
			return this.name.compareTo(s.name);
		return num;
	}

	public String getName()
	{
		return name;
	}
	
	public int getSum()
	{
		return sum;
	}

	public int hashCode()
	{
		return name.hashCode()+sum*78;
	}
	
	public boolean equals(Object obj)
	{
		if(!(obj instanceof Student))
			throw new ClassCastException("类型不匹配");
		Student s = (Student)obj;
		return this.name.equals(s.name) && this.sum==s.sum;
	}

	public String toString()
	{
		return "student["+name+", "+ma+", "+cn+", "+en+"]";
	}
}

class StudentInfoTool
{
	public static Set<Student> getStudents()throws IOException
	{
		return getStudents(null);
	}

	public static Set<Student> getStudents(Comparator<Student> cmp)throws IOException
	{
		BufferedReader bufr = 
			new BufferedReader(new InputStreamReader(System.in));

		String line = null;	
		Set<Student> stus  = null;
		if(cmp==null)
			stus = new TreeSet<Student>();
		else
			stus = new TreeSet<Student>(cmp);
		while((line=bufr.readLine())!=null)
		{
			if("over".equals(line))
				break;
			
			String[] info = line.split(",");
			
			Student stu = new Student(info[0],Integer.parseInt(info[1]),
										Integer.parseInt(info[2]),
										Integer.parseInt(info[3]));			
			stus.add(stu);
		}
		bufr.close();
		return stus;
	}

	public static void write2File(Set<Student> stus)throws IOException
	{
		BufferedWriter bufw = new BufferedWriter(new FileWriter("stuinfo.txt"));

		for(Student stu : stus)
		{
			bufw.write(stu.toString()+"\t");
			bufw.write(stu.getSum()+"");
			bufw.newLine();
			bufw.flush();
		}
		bufw.close();
	}
}

class StudentInfoTest 
{
	public static void main(String[] args) throws IOException
	{

		Comparator<Student> cmp = Collections.reverseOrder();

		Set<Student> stus = StudentInfoTool.getStudents(cmp);

		StudentInfoTool.write2File(stus);
	}
}
~~~



