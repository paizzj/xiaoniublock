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
  注意这里unique_ptr的成员函数reset: 带参数(如794行)和不带参数(p.reset)是有区别的,后者是清空拥有物.                      
  811行通过标准库中的函数make_shared分配类型为T的共享指针，并以参数deserialize,is传入.        
  注意 794和811这两处都调用了类型T的构造函数，794行通过new关键词调用，811行通过make_shared调用.
  
  通过以上分析就解释了40行为什么要加修饰符constexpr，是因为deserialize 要作为构造函数的参数传入，      
  而构造函数的参数一定要是个常量表达式(这里是猜测的，待验证TODO)      

  最后还有个疑问，deserialize的作用到底是啥呢？看20行的注释，就是用来区分反序列化对象的构造函数的，          
  那为什么要区分呢？ 这里只是一个约定，形如33行的声明的函数就约定是构造函数.


<pre><code>
    42	/**
    43	 * Used to bypass the rule against non-const reference to temporary
    44	 * where it makes sense with wrappers such as CFlatData or CTxDB
    45	 */
    46	template<typename T>
    47	inline T& REF(const T& val)
    48	{
    49	    return const_cast<T&>(val);
    50	}
    51	
    52	/**
    53	 * Used to acquire a non-const pointer "this" to generate bodies
    54	 * of const serialization operations from a template
    55	 */
    56	template<typename T>
    57	inline T* NCONST_PTR(const T* val)
    58	{
    59	    return const_cast<T*>(val);
    60	}
</code></pre>
* 42行到60行定义了2个模板函数，REF(), NCONST_PTR(),      
  前者主要是从const类型转换到非const的引用类型.      
  NCONST_PTR 是把const的指针类型转换为非const的指针类型.      
  这里涉及到常量引用和非常量引用的问题参考[const ref & nonconst ref][ref]

<pre><code>
    62	/*
    63	 * Lowest-level serialization and conversion.
    64	 * @note Sizes of these types are verified in the tests
    65	 */
    66	template<typename Stream> inline void ser_writedata8(Stream &s, uint8_t obj)
    67	{
    68	    s.write((char*)&obj, 1);
    69	}
    70	template<typename Stream> inline void ser_writedata16(Stream &s, uint16_t obj)
    71	{
    72	    obj = htole16(obj);
    73	    s.write((char*)&obj, 2);
    74	}
    75	template<typename Stream> inline void ser_writedata32(Stream &s, uint32_t obj)
    76	{
    77	    obj = htole32(obj);
    78	    s.write((char*)&obj, 4);
    79	}
    80	template<typename Stream> inline void ser_writedata64(Stream &s, uint64_t obj)
    81	{
    82	    obj = htole64(obj);
    83	    s.write((char*)&obj, 8);
    84	}
    85	template<typename Stream> inline uint8_t ser_readdata8(Stream &s)
    86	{
    87	    uint8_t obj;
    88	    s.read((char*)&obj, 1);
    89	    return obj;
    90	}
    91	template<typename Stream> inline uint16_t ser_readdata16(Stream &s)
    92	{
    93	    uint16_t obj;
    94	    s.read((char*)&obj, 2);
    95	    return le16toh(obj);
    96	}
    97	template<typename Stream> inline uint32_t ser_readdata32(Stream &s)
    98	{
    99	    uint32_t obj;
   100	    s.read((char*)&obj, 4);
   101	    return le32toh(obj);
   102	}
   103	template<typename Stream> inline uint64_t ser_readdata64(Stream &s)
   104	{
   105	    uint64_t obj;
   106	    s.read((char*)&obj, 8);
   107	    return le64toh(obj);
   108	}
   109	inline uint64_t ser_double_to_uint64(double x)
   110	{
   111	    union { double x; uint64_t y; } tmp;
   112	    tmp.x = x;
   113	    return tmp.y;
   114	}
   115	inline uint32_t ser_float_to_uint32(float x)
   116	{
   117	    union { float x; uint32_t y; } tmp;
   118	    tmp.x = x;
   119	    return tmp.y;
   120	}
   121	inline double ser_uint64_to_double(uint64_t y)
   122	{
   123	    union { double x; uint64_t y; } tmp;
   124	    tmp.y = y;
   125	    return tmp.x;
   126	}
   127	inline float ser_uint32_to_float(uint32_t y)
   128	{
   129	    union { float x; uint32_t y; } tmp;
   130	    tmp.y = y;
   131	    return tmp.x;
   132	}
</code></pre>
* 62行到132行是最底层的序列化转换函数.

<pre><code>
   135	/////////////////////////////////////////////////////////////////
   136	//
   137	// Templates for serializing to anything that looks like a stream,
   138	// i.e. anything that supports .read(char*, size_t) and .write(char*, size_t)
   139	//
   140	
   141	class CSizeComputer;
   142	
   143	enum
   144	{
   145	    // primary actions
   146	    SER_NETWORK         = (1 << 0),
   147	    SER_DISK            = (1 << 1),
   148	    SER_GETHASH         = (1 << 2),
   149	};
   150	
   151	#define READWRITE(obj)      (::SerReadWrite(s, (obj), ser_action))
   152	#define READWRITEMANY(...)      (::SerReadWriteMany(s, ser_action, __VA_ARGS__))
   153	
   154	/** 
   155	 * Implement three methods for serializable objects. These are actually wrappers over
   156	 * "SerializationOp" template, which implements the body of each class' serialization
   157	 * code. Adding "ADD_SERIALIZE_METHODS" in the body of the class causes these wrappers to be
   158	 * added as members. 
   159	 */
   160	#define ADD_SERIALIZE_METHODS                                         \
   161	    template<typename Stream>                                         \
   162	    void Serialize(Stream& s) const {                                 \
   163	        NCONST_PTR(this)->SerializationOp(s, CSerActionSerialize());  \
   164	    }                                                                 \
   165	    template<typename Stream>                                         \
   166	    void Unserialize(Stream& s) {                                     \
   167	        SerializationOp(s, CSerActionUnserialize());                  \
   168	    }
</code></pre>
* 141行声明类CSizeComputer
  143行定义枚举类型的序列化标志，SER_NETWORK: 网络， SER_DISK: 磁盘, SER_GETHASH: 哈希
  151,152行定义了2个宏，作用是具体的序列化操作
  154到168行，定义宏ADD_SERIALIZE_METHODS, 这个宏是嵌入到需要序列化的类中的,              
  注意155行的注释，描述说为序列化的对象实现了3个方法，分别是: Serialize, Unserialize, SerializationOp      
  SerializationOp 模板函数是相应的序列化的类必须实现的
  
<pre><code>
   170	template<typename Stream> inline void Serialize(Stream& s, char a    ) { ser_writedata8(s, a); } // TODO Get rid of bare char
   171	template<typename Stream> inline void Serialize(Stream& s, int8_t a  ) { ser_writedata8(s, a); }
   172	template<typename Stream> inline void Serialize(Stream& s, uint8_t a ) { ser_writedata8(s, a); }
   173	template<typename Stream> inline void Serialize(Stream& s, int16_t a ) { ser_writedata16(s, a); }
   174	template<typename Stream> inline void Serialize(Stream& s, uint16_t a) { ser_writedata16(s, a); }
   175	template<typename Stream> inline void Serialize(Stream& s, int32_t a ) { ser_writedata32(s, a); }
   176	template<typename Stream> inline void Serialize(Stream& s, uint32_t a) { ser_writedata32(s, a); }
   177	template<typename Stream> inline void Serialize(Stream& s, int64_t a ) { ser_writedata64(s, a); }
   178	template<typename Stream> inline void Serialize(Stream& s, uint64_t a) { ser_writedata64(s, a); }
   179	template<typename Stream> inline void Serialize(Stream& s, float a   ) { ser_writedata32(s, ser_float_to_uint32(a)); }
   180	template<typename Stream> inline void Serialize(Stream& s, double a  ) { ser_writedata64(s, ser_double_to_uint64(a)); }
   181	
   182	template<typename Stream> inline void Unserialize(Stream& s, char& a    ) { a = ser_readdata8(s); } // TODO Get rid of bare char
   183	template<typename Stream> inline void Unserialize(Stream& s, int8_t& a  ) { a = ser_readdata8(s); }
   184	template<typename Stream> inline void Unserialize(Stream& s, uint8_t& a ) { a = ser_readdata8(s); }
   185	template<typename Stream> inline void Unserialize(Stream& s, int16_t& a ) { a = ser_readdata16(s); }
   186	template<typename Stream> inline void Unserialize(Stream& s, uint16_t& a) { a = ser_readdata16(s); }
   187	template<typename Stream> inline void Unserialize(Stream& s, int32_t& a ) { a = ser_readdata32(s); }
   188	template<typename Stream> inline void Unserialize(Stream& s, uint32_t& a) { a = ser_readdata32(s); }
   189	template<typename Stream> inline void Unserialize(Stream& s, int64_t& a ) { a = ser_readdata64(s); }
   190	template<typename Stream> inline void Unserialize(Stream& s, uint64_t& a) { a = ser_readdata64(s); }
   191	template<typename Stream> inline void Unserialize(Stream& s, float& a   ) { a = ser_uint32_to_float(ser_readdata32(s)); }
   192	template<typename Stream> inline void Unserialize(Stream& s, double& a  ) { a = ser_uint64_to_double(ser_readdata64(s)); }
   193	
   194	template<typename Stream> inline void Serialize(Stream& s, bool a)    { char f=a; ser_writedata8(s, f); }
   195	template<typename Stream> inline void Unserialize(Stream& s, bool& a) { char f=ser_readdata8(s); a=f; }
</code></pre>
* 170行到195行定义了针对不同的基本类型的特化版本，底层实现是基于62行到132行定义的ser_writedataX(s,a)函数.

<pre><code>
   202	/**
   203	 * Compact Size
   204	 * size <  253        -- 1 byte
   205	 * size <= USHRT_MAX  -- 3 bytes  (253 + 2 bytes)
   206	 * size <= UINT_MAX   -- 5 bytes  (254 + 4 bytes)
   207	 * size >  UINT_MAX   -- 9 bytes  (255 + 8 bytes)
   208	 */
   209	inline unsigned int GetSizeOfCompactSize(uint64_t nSize)
   210	{
   211	    if (nSize < 253)             return sizeof(unsigned char);
   212	    else if (nSize <= std::numeric_limits<unsigned short>::max()) return sizeof(unsigned char) + sizeof(unsigned short);
   213	    else if (nSize <= std::numeric_limits<unsigned int>::max())  return sizeof(unsigned char) + sizeof(unsigned int);
   214	    else                         return sizeof(unsigned char) + sizeof(uint64_t);
   215	}
   216	
   217	inline void WriteCompactSize(CSizeComputer& os, uint64_t nSize);
   218	
   219	template<typename Stream>
   220	void WriteCompactSize(Stream& os, uint64_t nSize)
   221	{
   222	    if (nSize < 253)
   223	    {
   224	        ser_writedata8(os, nSize);
   225	    }
   226	    else if (nSize <= std::numeric_limits<unsigned short>::max())
   227	    {
   228	        ser_writedata8(os, 253);
   229	        ser_writedata16(os, nSize);
   230	    }
   231	    else if (nSize <= std::numeric_limits<unsigned int>::max())
   232	    {
   233	        ser_writedata8(os, 254);
   234	        ser_writedata32(os, nSize);
   235	    }
   236	    else
   237	    {
   238	        ser_writedata8(os, 255);
   239	        ser_writedata64(os, nSize);
   240	    }
   241	    return;
   242	}
   243	
   244	template<typename Stream>
   245	uint64_t ReadCompactSize(Stream& is)
   246	{
   247	    uint8_t chSize = ser_readdata8(is);
   248	    uint64_t nSizeRet = 0;
   249	    if (chSize < 253)
   250	    {
   251	        nSizeRet = chSize;
   252	    }
   253	    else if (chSize == 253)
   254	    {
   255	        nSizeRet = ser_readdata16(is);
   256	        if (nSizeRet < 253)
   257	            throw std::ios_base::failure("non-canonical ReadCompactSize()");
   258	    }
   259	    else if (chSize == 254)
   260	    {
   261	        nSizeRet = ser_readdata32(is);
   262	        if (nSizeRet < 0x10000u)
   263	            throw std::ios_base::failure("non-canonical ReadCompactSize()");
   264	    }
   265	    else
   266	    {
   267	        nSizeRet = ser_readdata64(is);
   268	        if (nSizeRet < 0x100000000ULL)
   269	            throw std::ios_base::failure("non-canonical ReadCompactSize()");
   270	    }
   271	    if (nSizeRet > (uint64_t)MAX_SIZE)
   272	        throw std::ios_base::failure("ReadCompactSize(): size too large");
   273	    return nSizeRet;
   274	}
</code></pre>
* 不同类型的数字的紧凑编码，以及相应的读写操作，我们会问这些读写函数怎么用呢？(TODO)              

<pre><code>
   276	/**
   277	 * Variable-length integers: bytes are a MSB base-128 encoding of the number.
   278	 * The high bit in each byte signifies whether another digit follows. To make
   279	 * sure the encoding is one-to-one, one is subtracted from all but the last digit.
   280	 * Thus, the byte sequence a[] with length len, where all but the last byte
   281	 * has bit 128 set, encodes the number:
   282	 * 
   283	 *  (a[len-1] & 0x7F) + sum(i=1..len-1, 128^i*((a[len-i-1] & 0x7F)+1))
   284	 * 
   285	 * Properties:
   286	 * * Very small (0-127: 1 byte, 128-16511: 2 bytes, 16512-2113663: 3 bytes)
   287	 * * Every integer has exactly one encoding
   288	 * * Encoding does not depend on size of original integer type
   289	 * * No redundancy: every (infinite) byte sequence corresponds to a list
   290	 *   of encoded integers.
   291	 * 
   292	 * 0:         [0x00]  256:        [0x81 0x00]
   293	 * 1:         [0x01]  16383:      [0xFE 0x7F]
   294	 * 127:       [0x7F]  16384:      [0xFF 0x00]
   295	 * 128:  [0x80 0x00]  16511:      [0xFF 0x7F]
   296	 * 255:  [0x80 0x7F]  65535: [0x82 0xFE 0x7F]
   297	 * 2^32:           [0x8E 0xFE 0xFE 0xFF 0x00]
   298	 */
   299	
   300	template<typename I>
   301	inline unsigned int GetSizeOfVarInt(I n)
   302	{
   303	    int nRet = 0;
   304	    while(true) {
   305	        nRet++;
   306	        if (n <= 0x7F)
   307	            break;
   308	        n = (n >> 7) - 1;
   309	    }
   310	    return nRet;
   311	}
   312	
   313	template<typename I>
   314	inline void WriteVarInt(CSizeComputer& os, I n);
   315	
   316	template<typename Stream, typename I>
   317	void WriteVarInt(Stream& os, I n)
   318	{
   319	    unsigned char tmp[(sizeof(n)*8+6)/7];
   320	    int len=0;
   321	    while(true) {
   322	        tmp[len] = (n & 0x7F) | (len ? 0x80 : 0x00);
   323	        if (n <= 0x7F)
   324	            break;
   325	        n = (n >> 7) - 1;
   326	        len++;
   327	    }
   328	    do {
   329	        ser_writedata8(os, tmp[len]);
   330	    } while(len--);
   331	}
   332	
   333	template<typename Stream, typename I>
   334	I ReadVarInt(Stream& is)
   335	{
   336	    I n = 0;
   337	    while(true) {
   338	        unsigned char chData = ser_readdata8(is);
   339	        if (n > (std::numeric_limits<I>::max() >> 7)) {
   340	           throw std::ios_base::failure("ReadVarInt(): size too large");
   341	        }
   342	        n = (n << 7) | (chData & 0x7F);
   343	        if (chData & 0x80) {
   344	            if (n == std::numeric_limits<I>::max()) {
   345	                throw std::ios_base::failure("ReadVarInt(): size too large");
   346	            }
   347	            n++;
   348	        } else {
   349	            return n;
   350	        }
   351	    }
   352	}
   353	
   354	#define FLATDATA(obj) REF(CFlatData((char*)&(obj), (char*)&(obj) + sizeof(obj)))
   355	#define VARINT(obj) REF(WrapVarInt(REF(obj)))
   356	#define COMPACTSIZE(obj) REF(CCompactSize(REF(obj)))
   357	#define LIMITED_STRING(obj,n) REF(LimitedString< n >(REF(obj)))
   358	
   359	/** 
   360	 * Wrapper for serializing arrays and POD.
   361	 */
   362	class CFlatData
   363	{
   364	protected:
   365	    char* pbegin;
   366	    char* pend;
   367	public:
   368	    CFlatData(void* pbeginIn, void* pendIn) : pbegin((char*)pbeginIn), pend((char*)pendIn) { }
   369	    template <class T, class TAl>
   370	    explicit CFlatData(std::vector<T,TAl> &v)
   371	    {
   372	        pbegin = (char*)v.data();
   373	        pend = (char*)(v.data() + v.size());
   374	    }
   375	    template <unsigned int N, typename T, typename S, typename D>
   376	    explicit CFlatData(prevector<N, T, S, D> &v)
   377	    {
   378	        pbegin = (char*)v.data();
   379	        pend = (char*)(v.data() + v.size());
   380	    }
   381	    char* begin() { return pbegin; }
   382	    const char* begin() const { return pbegin; }
   383	    char* end() { return pend; }
   384	    const char* end() const { return pend; }
   385	
   386	    template<typename Stream>
   387	    void Serialize(Stream& s) const
   388	    {
   389	        s.write(pbegin, pend - pbegin);
   390	    }
   391	
   392	    template<typename Stream>
   393	    void Unserialize(Stream& s)
   394	    {
   395	        s.read(pbegin, pend - pbegin);
   396	    }
   397	};
   398	
   399	template<typename I>
   400	class CVarInt
   401	{
   402	protected:
   403	    I &n;
   404	public:
   405	    explicit CVarInt(I& nIn) : n(nIn) { }
   406	
   407	    template<typename Stream>
   408	    void Serialize(Stream &s) const {
   409	        WriteVarInt<Stream,I>(s, n);
   410	    }
   411	
   412	    template<typename Stream>
   413	    void Unserialize(Stream& s) {
   414	        n = ReadVarInt<Stream,I>(s);
   415	    }
   416	};
   417	
   418	class CCompactSize
   419	{
   420	protected:
   421	    uint64_t &n;
   422	public:
   423	    explicit CCompactSize(uint64_t& nIn) : n(nIn) { }
   424	
   425	    template<typename Stream>
   426	    void Serialize(Stream &s) const {
   427	        WriteCompactSize<Stream>(s, n);
   428	    }
   429	
   430	    template<typename Stream>
   431	    void Unserialize(Stream& s) {
   432	        n = ReadCompactSize<Stream>(s);
   433	    }
   434	};
   435	
   436	template<size_t Limit>
   437	class LimitedString
   438	{
   439	protected:
   440	    std::string& string;
   441	public:
   442	    explicit LimitedString(std::string& _string) : string(_string) {}
   443	
   444	    template<typename Stream>
   445	    void Unserialize(Stream& s)
   446	    {
   447	        size_t size = ReadCompactSize(s);
   448	        if (size > Limit) {
   449	            throw std::ios_base::failure("String length limit exceeded");
   450	        }
   451	        string.resize(size);
   452	        if (size != 0)
   453	            s.read((char*)string.data(), size);
   454	    }
   455	
   456	    template<typename Stream>
   457	    void Serialize(Stream& s) const
   458	    {
   459	        WriteCompactSize(s, string.size());
   460	        if (!string.empty())
   461	            s.write((char*)string.data(), string.size());
   462	    }
   463	};
   464	
   465	template<typename I>
   466	CVarInt<I> WrapVarInt(I& n) { return CVarInt<I>(n); }
</code></pre>
* 276行到352行 主要是变长的整数处理.(TODO)       
  354行到357行 定义了4个在需要序列化的类中用的的宏          
  360行到397行 定义了针对序列化数组和POD类型的封装类CFlatData        
  399行到416行 定义了变长整数类CVarInt        
  418行到434行 定义了紧凑类CCompactSize        
  436行到463行 定义了受限制的字符串类 LimitedString     
  465,466行    定义了封装CVarInt的类 WrapVarInt      
  注意类CFloatData, CVarInt, CCompatcSize,LimitedString 这4个类都自己定义了序列化和反序列化方法.

<pre><code>
   468	/**
   469	 * Forward declarations
   470	 */
   471	
   472	/**
   473	 *  string
   474	 */
   475	template<typename Stream, typename C> void Serialize(Stream& os, const std::basic_string<C>& str);
   476	template<typename Stream, typename C> void Unserialize(Stream& is, std::basic_string<C>& str);
   477	
   478	/**
   479	 * prevector
   480	 * prevectors of unsigned char are a special case and are intended to be serialized as a single opaque blob.
   481	 */
   482	template<typename Stream, unsigned int N, typename T> void Serialize_impl(Stream& os, const prevector<N, T>& v, const unsigned char&);
   483	template<typename Stream, unsigned int N, typename T, typename V> void Serialize_impl(Stream& os, const prevector<N, T>& v, const V&);
   484	template<typename Stream, unsigned int N, typename T> inline void Serialize(Stream& os, const prevector<N, T>& v);
   485	template<typename Stream, unsigned int N, typename T> void Unserialize_impl(Stream& is, prevector<N, T>& v, const unsigned char&);
   486	template<typename Stream, unsigned int N, typename T, typename V> void Unserialize_impl(Stream& is, prevector<N, T>& v, const V&);
   487	template<typename Stream, unsigned int N, typename T> inline void Unserialize(Stream& is, prevector<N, T>& v);
   488	
   489	/**
   490	 * vector
   491	 * vectors of unsigned char are a special case and are intended to be serialized as a single opaque blob.
   492	 */
   493	template<typename Stream, typename T, typename A> void Serialize_impl(Stream& os, const std::vector<T, A>& v, const unsigned char&);
   494	template<typename Stream, typename T, typename A, typename V> void Serialize_impl(Stream& os, const std::vector<T, A>& v, const V&);
   495	template<typename Stream, typename T, typename A> inline void Serialize(Stream& os, const std::vector<T, A>& v);
   496	template<typename Stream, typename T, typename A> void Unserialize_impl(Stream& is, std::vector<T, A>& v, const unsigned char&);
   497	template<typename Stream, typename T, typename A, typename V> void Unserialize_impl(Stream& is, std::vector<T, A>& v, const V&);
   498	template<typename Stream, typename T, typename A> inline void Unserialize(Stream& is, std::vector<T, A>& v);
   499	
   500	/**
   501	 * pair
   502	 */
   503	template<typename Stream, typename K, typename T> void Serialize(Stream& os, const std::pair<K, T>& item);
   504	template<typename Stream, typename K, typename T> void Unserialize(Stream& is, std::pair<K, T>& item);
   505	
   506	/**
   507	 * map
   508	 */
   509	template<typename Stream, typename K, typename T, typename Pred, typename A> void Serialize(Stream& os, const std::map<K, T, Pred, A>& m);
   510	template<typename Stream, typename K, typename T, typename Pred, typename A> void Unserialize(Stream& is, std::map<K, T, Pred, A>& m);
   511	
   512	/**
   513	 * set
   514	 */
   515	template<typename Stream, typename K, typename Pred, typename A> void Serialize(Stream& os, const std::set<K, Pred, A>& m);
   516	template<typename Stream, typename K, typename Pred, typename A> void Unserialize(Stream& is, std::set<K, Pred, A>& m);
   517	
   518	/**
   519	 * shared_ptr
   520	 */
   521	template<typename Stream, typename T> void Serialize(Stream& os, const std::shared_ptr<const T>& p);
   522	template<typename Stream, typename T> void Unserialize(Stream& os, std::shared_ptr<const T>& p);
   523	
   524	/**
   525	 * unique_ptr
   526	 */
   527	template<typename Stream, typename T> void Serialize(Stream& os, const std::unique_ptr<const T>& p);
   528	template<typename Stream, typename T> void Unserialize(Stream& os, std::unique_ptr<const T>& p);
   529	
   530	
   531	
   532	/**
   533	 * If none of the specialized versions above matched, default to calling member function.
   534	 */
   535	template<typename Stream, typename T>
   536	inline void Serialize(Stream& os, const T& a)
   537	{
   538	    a.Serialize(os);
   539	}
   540	
   541	template<typename Stream, typename T>
   542	inline void Unserialize(Stream& is, T& a)
   543	{
   544	    a.Unserialize(is);
   545	}
   546	
   547	
   548	
   549	
   550	
   551	/**
   552	 * string
   553	 */
   554	template<typename Stream, typename C>
   555	void Serialize(Stream& os, const std::basic_string<C>& str)
   556	{
   557	    WriteCompactSize(os, str.size());
   558	    if (!str.empty())
   559	        os.write((char*)str.data(), str.size() * sizeof(C));
   560	}
   561	
   562	template<typename Stream, typename C>
   563	void Unserialize(Stream& is, std::basic_string<C>& str)
   564	{
   565	    unsigned int nSize = ReadCompactSize(is);
   566	    str.resize(nSize);
   567	    if (nSize != 0)
   568	        is.read((char*)str.data(), nSize * sizeof(C));
   569	}
   570	
   571	
   572	
   573	/**
   574	 * prevector
   575	 */
   576	template<typename Stream, unsigned int N, typename T>
   577	void Serialize_impl(Stream& os, const prevector<N, T>& v, const unsigned char&)
   578	{
   579	    WriteCompactSize(os, v.size());
   580	    if (!v.empty())
   581	        os.write((char*)v.data(), v.size() * sizeof(T));
   582	}
   583	
   584	template<typename Stream, unsigned int N, typename T, typename V>
   585	void Serialize_impl(Stream& os, const prevector<N, T>& v, const V&)
   586	{
   587	    WriteCompactSize(os, v.size());
   588	    for (typename prevector<N, T>::const_iterator vi = v.begin(); vi != v.end(); ++vi)
   589	        ::Serialize(os, (*vi));
   590	}
   591	
   592	template<typename Stream, unsigned int N, typename T>
   593	inline void Serialize(Stream& os, const prevector<N, T>& v)
   594	{
   595	    Serialize_impl(os, v, T());
   596	}
   597	
   598	
   599	template<typename Stream, unsigned int N, typename T>
   600	void Unserialize_impl(Stream& is, prevector<N, T>& v, const unsigned char&)
   601	{
   602	    // Limit size per read so bogus size value won't cause out of memory
   603	    v.clear();
   604	    unsigned int nSize = ReadCompactSize(is);
   605	    unsigned int i = 0;
   606	    while (i < nSize)
   607	    {
   608	        unsigned int blk = std::min(nSize - i, (unsigned int)(1 + 4999999 / sizeof(T)));
   609	        v.resize(i + blk);
   610	        is.read((char*)&v[i], blk * sizeof(T));
   611	        i += blk;
   612	    }
   613	}
   614	
   615	template<typename Stream, unsigned int N, typename T, typename V>
   616	void Unserialize_impl(Stream& is, prevector<N, T>& v, const V&)
   617	{
   618	    v.clear();
   619	    unsigned int nSize = ReadCompactSize(is);
   620	    unsigned int i = 0;
   621	    unsigned int nMid = 0;
   622	    while (nMid < nSize)
   623	    {
   624	        nMid += 5000000 / sizeof(T);
   625	        if (nMid > nSize)
   626	            nMid = nSize;
   627	        v.resize(nMid);
   628	        for (; i < nMid; i++)
   629	            Unserialize(is, v[i]);
   630	    }
   631	}
   632	
   633	template<typename Stream, unsigned int N, typename T>
   634	inline void Unserialize(Stream& is, prevector<N, T>& v)
   635	{
   636	    Unserialize_impl(is, v, T());
   637	}
   638	
   639	
   640	
   641	/**
   642	 * vector
   643	 */
   644	template<typename Stream, typename T, typename A>
   645	void Serialize_impl(Stream& os, const std::vector<T, A>& v, const unsigned char&)
   646	{
   647	    WriteCompactSize(os, v.size());
   648	    if (!v.empty())
   649	        os.write((char*)v.data(), v.size() * sizeof(T));
   650	}
   651	
   652	template<typename Stream, typename T, typename A, typename V>
   653	void Serialize_impl(Stream& os, const std::vector<T, A>& v, const V&)
   654	{
   655	    WriteCompactSize(os, v.size());
   656	    for (typename std::vector<T, A>::const_iterator vi = v.begin(); vi != v.end(); ++vi)
   657	        ::Serialize(os, (*vi));
   658	}
   659	
   660	template<typename Stream, typename T, typename A>
   661	inline void Serialize(Stream& os, const std::vector<T, A>& v)
   662	{
   663	    Serialize_impl(os, v, T());
   664	}
   665	
   666	
   667	template<typename Stream, typename T, typename A>
   668	void Unserialize_impl(Stream& is, std::vector<T, A>& v, const unsigned char&)
   669	{
   670	    // Limit size per read so bogus size value won't cause out of memory
   671	    v.clear();
   672	    unsigned int nSize = ReadCompactSize(is);
   673	    unsigned int i = 0;
   674	    while (i < nSize)
   675	    {
   676	        unsigned int blk = std::min(nSize - i, (unsigned int)(1 + 4999999 / sizeof(T)));
   677	        v.resize(i + blk);
   678	        is.read((char*)&v[i], blk * sizeof(T));
   679	        i += blk;
   680	    }
   681	}
   682	
   683	template<typename Stream, typename T, typename A, typename V>
   684	void Unserialize_impl(Stream& is, std::vector<T, A>& v, const V&)
   685	{
   686	    v.clear();
   687	    unsigned int nSize = ReadCompactSize(is);
   688	    unsigned int i = 0;
   689	    unsigned int nMid = 0;
   690	    while (nMid < nSize)
   691	    {
   692	        nMid += 5000000 / sizeof(T);
   693	        if (nMid > nSize)
   694	            nMid = nSize;
   695	        v.resize(nMid);
   696	        for (; i < nMid; i++)
   697	            Unserialize(is, v[i]);
   698	    }
   699	}
   700	
   701	template<typename Stream, typename T, typename A>
   702	inline void Unserialize(Stream& is, std::vector<T, A>& v)
   703	{
   704	    Unserialize_impl(is, v, T());
   705	}
   706	
   707	
   708	
   709	/**
   710	 * pair
   711	 */
   712	template<typename Stream, typename K, typename T>
   713	void Serialize(Stream& os, const std::pair<K, T>& item)
   714	{
   715	    Serialize(os, item.first);
   716	    Serialize(os, item.second);
   717	}
   718	
   719	template<typename Stream, typename K, typename T>
   720	void Unserialize(Stream& is, std::pair<K, T>& item)
   721	{
   722	    Unserialize(is, item.first);
   723	    Unserialize(is, item.second);
   724	}
   725	
   726	
   727	
   728	/**
   729	 * map
   730	 */
   731	template<typename Stream, typename K, typename T, typename Pred, typename A>
   732	void Serialize(Stream& os, const std::map<K, T, Pred, A>& m)
   733	{
   734	    WriteCompactSize(os, m.size());
   735	    for (typename std::map<K, T, Pred, A>::const_iterator mi = m.begin(); mi != m.end(); ++mi)
   736	        Serialize(os, (*mi));
   737	}
   738	
   739	template<typename Stream, typename K, typename T, typename Pred, typename A>
   740	void Unserialize(Stream& is, std::map<K, T, Pred, A>& m)
   741	{
   742	    m.clear();
   743	    unsigned int nSize = ReadCompactSize(is);
   744	    typename std::map<K, T, Pred, A>::iterator mi = m.begin();
   745	    for (unsigned int i = 0; i < nSize; i++)
   746	    {
   747	        std::pair<K, T> item;
   748	        Unserialize(is, item);
   749	        mi = m.insert(mi, item);
   750	    }
   751	}
   752	
   753	
   754	
   755	/**
   756	 * set
   757	 */
   758	template<typename Stream, typename K, typename Pred, typename A>
   759	void Serialize(Stream& os, const std::set<K, Pred, A>& m)
   760	{
   761	    WriteCompactSize(os, m.size());
   762	    for (typename std::set<K, Pred, A>::const_iterator it = m.begin(); it != m.end(); ++it)
   763	        Serialize(os, (*it));
   764	}
   765	
   766	template<typename Stream, typename K, typename Pred, typename A>
   767	void Unserialize(Stream& is, std::set<K, Pred, A>& m)
   768	{
   769	    m.clear();
   770	    unsigned int nSize = ReadCompactSize(is);
   771	    typename std::set<K, Pred, A>::iterator it = m.begin();
   772	    for (unsigned int i = 0; i < nSize; i++)
   773	    {
   774	        K key;
   775	        Unserialize(is, key);
   776	        it = m.insert(it, key);
   777	    }
   778	}
   779	
   780	
   781	
   782	/**
   783	 * unique_ptr
   784	 */
   785	template<typename Stream, typename T> void
   786	Serialize(Stream& os, const std::unique_ptr<const T>& p)
   787	{
   788	    Serialize(os, *p);
   789	}
   790	
   791	template<typename Stream, typename T>
   792	void Unserialize(Stream& is, std::unique_ptr<const T>& p)
   793	{
   794	    p.reset(new T(deserialize, is));
   795	}
   796	
   797	
   798	
   799	/**
   800	 * shared_ptr
   801	 */
   802	template<typename Stream, typename T> void
   803	Serialize(Stream& os, const std::shared_ptr<const T>& p)
   804	{
   805	    Serialize(os, *p);
   806	}
   807	
   808	template<typename Stream, typename T>
   809	void Unserialize(Stream& is, std::shared_ptr<const T>& p)
   810	{
   811	    p = std::make_shared<const T>(deserialize, is);
   812	}
</code></pre>
* 468行到812行声明和定义了针对不同类型的特化版本和通用版本.

<pre><code>
   816	/**
   817	 * Support for ADD_SERIALIZE_METHODS and READWRITE macro
   818	 */
   819	struct CSerActionSerialize
   820	{
   821	    constexpr bool ForRead() const { return false; }
   822	};
   823	struct CSerActionUnserialize
   824	{
   825	    constexpr bool ForRead() const { return true; }
   826	};
   827	
   828	template<typename Stream, typename T>
   829	inline void SerReadWrite(Stream& s, const T& obj, CSerActionSerialize ser_action)
   830	{
   831	    ::Serialize(s, obj);
   832	}
   833	
   834	template<typename Stream, typename T>
   835	inline void SerReadWrite(Stream& s, T& obj, CSerActionUnserialize ser_action)
   836	{
   837	    ::Unserialize(s, obj);
   838	}
</code></pre>
* 定义了ADD_SERIALIZE_METHODS 和 READWRITE 2个宏用到的类和模板函数.


serialize.h 文件的最后一部分:            
<pre><code>
   848	/* ::GetSerializeSize implementations
   849	 *
   850	 * Computing the serialized size of objects is done through a special stream
   851	 * object of type CSizeComputer, which only records the number of bytes written
   852	 * to it.
   853	 *
   854	 * If your Serialize or SerializationOp method has non-trivial overhead for
   855	 * serialization, it may be worthwhile to implement a specialized version for
   856	 * CSizeComputer, which uses the s.seek() method to record bytes that would
   857	 * be written instead.
   858	 */
   859	class CSizeComputer
   860	{
   861	protected:
   862	    size_t nSize;
   863	
   864	    const int nType;
   865	    const int nVersion;
   866	public:
   867	    CSizeComputer(int nTypeIn, int nVersionIn) : nSize(0), nType(nTypeIn), nVersion(nVersionIn) {}
   868	
   869	    void write(const char *psz, size_t _nSize)
   870	    {
   871	        this->nSize += _nSize;
   872	    }
   873	
   874	    /** Pretend _nSize bytes are written, without specifying them. */
   875	    void seek(size_t _nSize)
   876	    {
   877	        this->nSize += _nSize;
   878	    }
   879	
   880	    template<typename T>
   881	    CSizeComputer& operator<<(const T& obj)
   882	    {
   883	        ::Serialize(*this, obj);
   884	        return (*this);
   885	    }
   886	
   887	    size_t size() const {
   888	        return nSize;
   889	    }
   890	
   891	    int GetVersion() const { return nVersion; }
   892	    int GetType() const { return nType; }
   893	};
   894	
   895	template<typename Stream>
   896	void SerializeMany(Stream& s)
   897	{
   898	}
   899	
   900	template<typename Stream, typename Arg>
   901	void SerializeMany(Stream& s, Arg&& arg)
   902	{
   903	    ::Serialize(s, std::forward<Arg>(arg));
   904	}
   905	
   906	template<typename Stream, typename Arg, typename... Args>
   907	void SerializeMany(Stream& s, Arg&& arg, Args&&... args)
   908	{
   909	    ::Serialize(s, std::forward<Arg>(arg));
   910	    ::SerializeMany(s, std::forward<Args>(args)...);
   911	}
   912	
   913	template<typename Stream>
   914	inline void UnserializeMany(Stream& s)
   915	{
   916	}
   917	
   918	template<typename Stream, typename Arg>
   919	inline void UnserializeMany(Stream& s, Arg& arg)
   920	{
   921	    ::Unserialize(s, arg);
   922	}
   923	
   924	template<typename Stream, typename Arg, typename... Args>
   925	inline void UnserializeMany(Stream& s, Arg& arg, Args&... args)
   926	{
   927	    ::Unserialize(s, arg);
   928	    ::UnserializeMany(s, args...);
   929	}
   930	
   931	template<typename Stream, typename... Args>
   932	inline void SerReadWriteMany(Stream& s, CSerActionSerialize ser_action, Args&&... args)
   933	{
   934	    ::SerializeMany(s, std::forward<Args>(args)...);
   935	}
   936	
   937	template<typename Stream, typename... Args>
   938	inline void SerReadWriteMany(Stream& s, CSerActionUnserialize ser_action, Args&... args)
   939	{
   940	    ::UnserializeMany(s, args...);
   941	}
   942	
   943	template<typename I>
   944	inline void WriteVarInt(CSizeComputer &s, I n)
   945	{
   946	    s.seek(GetSizeOfVarInt<I>(n));
   947	}
   948	
   949	inline void WriteCompactSize(CSizeComputer &s, uint64_t nSize)
   950	{
   951	    s.seek(GetSizeOfCompactSize(nSize));
   952	}
   953	
   954	template <typename T>
   955	size_t GetSerializeSize(const T& t, int nType, int nVersion = 0)
   956	{
   957	    return (CSizeComputer(nType, nVersion) << t).size();
   958	}
   959	
   960	template <typename S, typename T>
   961	size_t GetSerializeSize(const S& s, const T& t)
   962	{
   963	    return (CSizeComputer(s.GetType(), s.GetVersion()) << t).size();
   964	}
</code></pre>
* 859行到893行定义了类CSizeComputer,主要是为了实现方法GetSerializeSize().
* 893行到964行 主要实现了序列化的多个参数版本(many结尾的函数).


# 序列化总结 (TODO)

[ref]:https://stackoverflow.com/questions/1565600/how-come-a-non-const-reference-cannot-bind-to-a-temporary-object
[ser_src]:https://github.com/bitcoin/bitcoin/blob/master/src/serialize.h
[serialize]:http://blog.csdn.net/kiritow/article/details/53129096