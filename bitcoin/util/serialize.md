# serialization (序列化)
这里我们首先要明白，序列化的作用是什么? [可以参考序列化][serialize]    

* 这里简单整理下,序列化是指将数据从有结构清晰的语言定义的数据形式转化为二进制字符串，        
  反序列化则是序列化的逆操作。                  

* 百度百科定义序列化如下：      
序列化 (Serialization)将对象的状态信息转换为可以存储或传输的形式的过程。在序列化期间，对象将其当前状态写入        
到临时或持久性存储区。以后，可以通过从存储区中读取或反序列化对象的状态，重新创建该对象。           

* 在bitcoin项目中，序列化相关的文件: [serialize.h][ser_src]              
  这个文件定义bitcoin项目中所有类的序列化操作，个别类除外,比如类CAddrMan是单独写了序列化操作。


src/serialize.h :    
<pre><code>
    26	static const unsigned int MAX_SIZE = 0x02000000;     
    27	
    28	/**
    29	 * Dummy data type to identify deserializing constructors.
    30	 *
    31	 * By convention, a constructor of a type T with signature
    32	 *
    33	 *   template <typename Stream> T::T(deserialize_type, Stream& s)
    34	 *
    35	 * is a deserializing constructor, which builds the type by
    36	 * deserializing it from s. If T contains const fields, this
    37	 * is likely the only way to do so.
    38	 */
    39	struct deserialize_type {};
    40	constexpr deserialize_type deserialize {};
</code></pre>

* 第26到40行, 首先定义了常量MAX_SIZE,为后文作常量比较用. 接着是定义识别反序列化构造函数的空的数据类型.         
  28到38行的注释，完整的意思是: 类型T的构造函数是形如33行的声明,它通过从流s反序列化来构造类型，如果类型T      
  包含有const 字段，形如33行的是唯一的方式声明构造函数. 这个注释看得似懂非懂(TODO)      
  39行定义空的类(用struct定义)deserialize_type,没有任何成员的类.             
  40行依然定义空的类，只不过签名增加了constexpr修饰(c++11新增语法)，表示编译器会在编译的时候认为是一个常量表达式.     
  
* 当我们搜索文件serialize.h,发现用到deserialize这个类型的地方就两处：
<pre><code>
   791	template<typename Stream, typename T>
   792	void Unserialize(Stream& is, std::unique_ptr<const T>& p)
   793	{
   794	    p.reset(new T(deserialize, is));
   795	}
   ...... 
   ......
   808	template<typename Stream, typename T>
   809	void Unserialize(Stream& is, std::shared_ptr<const T>& p)
   810	{
   811	    p = std::make_shared<const T>(deserialize, is);
   812	}
</code></pre>
- 以上两处的模板函数定义是关于反序列化的unique_ptr 和shared_ptr的特化版本，          
  那对照之前第39,40行定义的空类型deserialize，对以上2个特化版本的模板函数又是什么用呢？(TODO)         
  794行通过new调用类型T的构造函数，并传入参数deserialize, 和流is,然后unique_ptr的reset,让p拥有T.      
  811行通过标准库中的函数make_shared分配类型为T的共享指针，并以参数deserialize,is传入.    
  注意 794和811这两处都调用了类型T的构造函数，794行通过new关键词调用，811行通过make_shared调用.
  
  通过以上分析就解释了40行为什么要加修饰符constexpr，是因为deserialize 要作为构造函数的参数传入，      
  而构造函数的参数一定要是个常量表达式(这里是猜测的，待验证TODO)      

  最后还有个疑问，deserialize的作用到底是啥呢？看20行的注释，就是用来区分反序列化对象的构造函数的，          
  那为什么要区分呢？ 这里只是一个约定，形如33行的声明的函数就约定是构造函数.















[ser_src]:https://github.com/bitcoin/bitcoin/blob/master/src/serialize.h
[serialize]:http://blog.csdn.net/kiritow/article/details/53129096