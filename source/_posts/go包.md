---
title: go包
date: 2023-02-03 14:03:57
tags: "go"
categories: "go"
---

### Archive

#### tar(实现tar格式压缩文件的存取)

```go
/*
Constants
Variables
type Format
	func (f Format) String() string
type Header
	func FileInfoHeader(fi fs.FileInfo,link string) (*Header,error)
	func (h *Header) FileInfo() fs.FileInfo
type Reader
	func NewReader(r io.Reader) *Reader
	func (tr *Reader) Next() (*Header,error)
	func (tr *Reader) Read(b []byte) (int,error)
type Writer
	func NewWriter(w io.Writer) *Writer
	func (tw *Writer) Close() error
	func (tw *Writer) Flush() error
	func (tw *Writer) Write(b []byte) (int,error)
	func (tw *Writer) WriteHeader(hdr *Header) error
*/
```

##### Header：表示tar文件里单个头

**FileInfoHeader(fi fs.FileInfo,link string) (*Header,error)**

返回一个根据fi填写了部分字段的Header，若fi描述了一个符号链接，则函数将link参数作为链接目标，若fi描述一个目录，则在名字后加斜杠

##### Reader：提供对tar档案文件的顺序读取

**NewReader(r io.Reader) *Reader**

创建一个从r读取的Reader

**(tr *Reader) Next() (\*Header,error)**

转入tar档案文件下一记录，返回下一记录的头域

**(tr *Reader) Read(b []byte) (n int,err error)**

从档案文件当前记录读取数据到记录末端返回(0,EOF)，直到调用Next转下一记录

##### Writer：提供POSIX.1格式的tar档案文件顺序写入

**NewWriter(w io.Writer) *Writer**

创建一个写入w的*Writer

**(tw *Writer) WriterHeader(hdr *Header) error**

写入hdr并准备接受文件内容，Header.Size取决于下一个文件能写入多少字节，若非首次调用本方法会调用Flush，在close后调用会返回ErrWriterAfterClose

**(tw *Writer) Write(b []byte) (n int,err error)**

Write向tar档案文件的当前记录中写入数据，若写入数据总数超出上一次调用WriteHeader的参数hdr.Size字节，返回ErrWriteTooLong错误

**(tw *Writer) Flush() error**

结束当前文件的写入

**(tw *Writer) Close() error**

关闭tar档案文件，将缓冲中未写入下层的io.Writer接口的数据刷新到下层

```go
func main(){
    //FileInfoHeader
    //func FileInfoHeader(fi fs.FileInfo,link string) (*Header,error)
    fileinfo,err := os.Stat("test.txt")
    if err != nil {
    	fmt.Println(err)
    }
    h,err := tar.FileInfoHeader(fileinfo,"")
    h.Linkname = "haha"
    h.Gname = "test"
    if err != nil {
    	fmt.Println(err)
    }
    fmt.Println(h.AccessTime,h.ChangeTime, h.Devmajor, h.Devminor, h.Gid,  h.Gname, h.Linkname, h.ModTime, h.Mode, h.Name, h.Size,h.Typeflag, h.Uid, h.Uname, h.Xattrs)
    
    //FileInfo返回Header对应的文件信息
    //func (h *Header) FileInfo() fs.FileInfo
    f,err := os.Open("sdk/test.tar")
    if err != nil {
		fmt.Println(err)
		return
	}
	defer f.Close()
		
    r := tar.NewReader(f)
	for hdr,err := r.Next(); err != io.EOF; hdr,err = r.Next() {
		if err != nil {
			fmt.Println(err)
			return
		}
			
		fileInfo := hdr.FileInfo()
		fmt.Println(fileInfo.Name())
	}
    
    //转入tar档案文件下一记录，返回下一记录的头域
    //func (tr *Reader) Next() (*Header,error)
    //func (tr *Reader) Read(b []byte) (int,error)
    r := tar.NewReader(strings.NewReader("test.tar"))
    h,err := r.Next()
    fmt.Println(h,err)
    
    n,err := tar.NewReader(strings.NewReader("test.log")).Read(make([]byte,0))
    fmt.Println(n,err)
    
    //Close关闭tar档案文件，会将缓冲中未写入下层的io.Writer接口的数据刷新到下层
    //func (tw *Writer) Close() error
    f,err := os.Create("sdk/demo.tar")
    if err != nil {
    	fmt.Println(err)
    	return
    }
    defer f.Close()
    tw := tar.NewWriter(f)
    defer tw.Close()
}
```

#### Zip

```go
/*
Constants
Variables
	func RegisterCompressor(method uint16,comp Compressor)
	func RegisterDecompressor(method unit16,dcomp Decompressor)
	type Compressor 
	type Decompressor
	type File
		func (f *File) DataOffset() (offset int64,err error)
		func (f *File) Open() (io.ReadCloser,error)
	type FileHeader
		func FileInfoHeader(fi fs.FileInfo) (*FileHeader,error)
		func (h *FileHeader) FileInfo() fs.FileInfo
		func (h *FileHeader) ModTime() time.Time
		func (h *FileHeader) Mode() (mode fs.FileMode)
		func (h *FileHeader) SetModTime(t time.Time)
		func (h *FileHeader) SetMode(mode fs.FileMode)
	type ReadCloser
		func OpenReader(name string) (*ReadCloser,error)
		func (rc *ReadCloser) Close() error
	type Reader
		func NewReader(r io.ReaderAt,size int64) (*Reader,error)
		func (r *Reader) Open(name string) (fs.File,error)
		func (z *Reader) RegisterDecompressor(method unit16,dcomp Decompressor)
	type Writer
		func NewWriter(w io.Writer) *Writer
		func (w *Writer) Close() error
		func (w *Writer) Crteate(name string) (io.Writer,error)
		func (w *Writer) CreateHeader(fh *FileHeader) (io.Writer,error)
		func (w *Writer) Flush() error
		func (w *Writer) RegisterCompressor(method unit16,comp Compressor)
		func (w *Writer) SetComment(comment string) error
		func (w *Writer) SetOffset(n int64)
*/
```

##### 压缩文件

###### 读取文件内容并压缩

```go
func CompressedFile(file *os.File,prefix string,zw *zip.Writer) error {
    info,err := file.Stat()
    if err != nil || info.IsDir() {
        return err
    }
    header,err := zip.FileInfoHeader(info)
    if err != nil {
        return err
    }
    header.Name = prefix + "/" + header.Name
    writer,err := zw.CreateHeader(header)
    if err != nil {
        return err
    }
    if _,err = io.Copy(writer,file); err != nil {
        return err
    }
    return nil
}

func main() {
    f,_ := os.Open("test.txt")
    //压缩文件
    dst,_ := os.Create("test.zip")
    zipWriter := zip.NewWriter(dst)
    if err := CompressFile(f,"",zipWriter); err != nil {
        log.Fatalln(err.Error())
    }
    // Make sure to check the error on Close.
    if err := zipWriter.Close(); err != nil {
        log.Fatalln(err.Error())
    }
    return
}
```

CompressedFile是将文件内容进行压缩，若想将目录进行压缩则将目录中的文件提取出来然后调用CompressedFile将文件压缩并写入zip.Writer即可：

```go
//Compress 压缩文件
func Compress(file *os.File, prefix string, zw *zip.Writer) error {
    info,err := file.Stat()
    if err != nil {
        return err
    }
    // 若是目录调用CompressedDir
    if info.IsDir() {
        return CompressedDir(file, prefix, zw)
    }
    // 若是文件调用CompressedFile
    return CompressedFile(file, prefix, zw)
}

//CompressedDir
func CompressedDir(file *os.File,prefix string,zw *zip.Writer) error {
    info,_ := file.Stat()
    prefix = prefix + "/" + info.Name()
    dirInfo,err := file.Readdir(-1)
    if err != nil {
        return err
    }
    for _,f := range dirInfo {
        f,err := os.Open(file.Name() + "/" + f.Name())
        if err != nil {
            return err
        }
        err = Compress(f,prefix,zw)
        if err != nil {
            return err
        }
    }
    return nil
}
```

###### 将数据直接写入压缩文件

将格式化数据写入压缩文件让用户下载，只对数据进行压缩不在本地生成文件：

```go
func CompressedData(data *bytes.Buffer,dest string) error {
    zipBuffer := new(bytes.Buffer)
    zipWriter := zip.NewWriter(zipBuffer)
    // Create entry in zip file
    zipEntry,err := zipWriter.Create(dest)
    if err != nil {
		return err
	}
    // Write content into zip writer
    if _,err := zipEntry.Write(data.Bytes());err != nil {
        return err
    }
    // Make sure to check the error on Close.
    if err := zipWriter.Close(); err != nil {
        return err
    }
    return nil
}
```

###### 解压缩文件

DeCompressed负责读取压缩文件并调用deCompressed，将读取的内容写入解压缩后的文件：

```go
func DeCompressed(src string) error {
    s,_ := os.Open(src)
    info,_ := s.Stat()
    ZipReader,err := zip.NewReader(s,info.Size())
    if err != nil {
        return err
    }
    for _,f := range ZipReader.File {
        if err := deCompressed(f); err != nil {
            return err
        }
    }
    return nil
}

func deCompressed(f *zip.File) error {
    d, _ := os.Create(f.Name)
    unzipFile,err := f.Open()
    if err != nil {
		return err
	}
    if _,err := io.Copy(d.unzipFile); err != nil {
        return err
    }
    if err := unzipFile.Close(); err != nil {
        return err
    }
    return d.Close()
}
```

### Bufio（实现有缓冲的I/O）

主要提供一些操作io缓存流的函数和方法，有时为了提高针对流操作的效率，不然不能缺少针对缓冲流的相关操作

```go
/*
	func ScanBytes(data []byte, atEOF bool) (advance int,token []byte,err error)
	func ScanLines(data []byte, atEOF bool) (advance int,token []byte,err error)
	func ScanRunes(data []byte, atEOF bool) (advance int,token []byte,err error)
	func ScanWords(data []byte, atEOF bool) (advance int,token []byte,err error)
type ReadWriter
	func NewReadWriter(r *Reader,w *Writer) *ReadWriter
type Reader
	func NewReader(rd io.Reader) *Reader
	func NewReaderSize(rd io.Reader,size int) *Reader
	func (b *Reader) Buffered() int
	func (b *Reader) Discard(n int) (discarded int,err error)
	func (b *Reader) Peek(n int) ([]byte,error)
	func (b *Reader) Read(p []byte) (n int, err error)
	func (b *Reader) ReadByte() (byte, error)
	func (b *Reader) ReadBytes(delim byte) ([]byte, error)
	func (b *Reader) ReadLine() (line []byte, isPrefix bool, err error)
	func (b *Reader) ReadRune() (r rune, size int, err error)
	func (b *Reader) ReadSlice(delim byte) (line []byte, err error)
	func (b *Reader) ReadString(delim byte) (string, error)
	func (b *Reader) Reset(r io.Reader)
	func (b *Reader) Size() int
	func (b *Reader) UnreadByte() error
	func (b *Reader) UnreadRune() error
	func (b *Reader) WriteTo(w io.Writer) (n int64, err error)
type Scanner
	func NewScanner(r io.Reader) *Scanner
	func (s *Scanner) Buffer(buf []byte,max int)
	func (s *Scanner) Bytes() []byte
	func (s *Scanner) Err() error
	func (s *Scanner) Scan() bool
	func (s *Scanner) Split(split SplitFunc)
	func (s *Scanner) Text() string
type SplitFunc
type Writer
	func NewWriter(w io.Writer) *Writer
	func NewWriterSize(w io.Writer,size int) *Writer
	func (b *Writer) Available() int
	func (b *Writer) Buffered() int
	func (b *Writer) Flush() error
	func (b *Writer) ReadFrom(r io.Reader) (n int64, err error)
	func (b *Writer) Reset(w io.Writer)
	func (b *Writer) Size() int
	func (b *Writer) Write(p []byte) (nn int, err error)
	func (b *Writer) WriteByte(c byte) error
	func (b *Writer) WriteRune(r rune) (size int, err error)
	func (b *Writer) WriteString(s string) (int, error)
*/
```

```go
func main() {
	/*
		ScanBytes逐字节读取数据
	*func ScanBytes(data []byte, atEOF bool) (advance int, token []byte, err error)
	*/
	s := strings.NewReader("Hello world")
	bs := bufio.NewScanner(s)
	bs.Split(bufio.ScanBytes)
	for bs.Scan() {
		fmt.Printf("%s\n", bs.Text())
	}
    
    /*
    	ScanLines逐行读取数据
	*func ScanLines(data []byte, atEOF bool) (advance int, token []byte, err error)
	*/
	s := strings.NewReader("Hello 世界\nhaha")
	bs := bufio.NewScanner(s)
	bs.Split(bufio.ScanLines)
	for bs.Scan() {
		fmt.Printf("%s\n", bs.Text())
	}
    
    /*
    	ScanRunes逐字读取数据
	*func ScanRunes(data []byte,atEOF bool) (advance int,token []byte,err error)
	*/
	s := strings.NewReader("hello 世界")
	bs := bufio.NewScanner(s)
	bs.Split(bufio.ScanRunes)
	for bs.Scan() {
		fmt.Printf("%s\n", bs.Text())
	}
    /*
    	输出：
    	h
    	e
    	l
    	l
    	o
    	
    	世
    	界
    */
    
    /*
    	ScanWords逐词读取数据
	*func ScanWords(data []byte,atEOF bool) (advance int,token []byte,err error)
	*/
	s := strings.NewReader("hello world! 世界 shijie sh")
	bs := bufio.NewScanner(s)
	bs.Split(bufio.ScanWords)
	for bs.Scan() {
		fmt.Printf("%s\n", bs.Text())
	}
    /*
    	输出:
    	hello
    	world!
    	世界
    	shijie
    	sh
    */
    
   	/*
   		NewReadWriter封装r和w为一个bufio。ReadWriter对象既可读也可写
   	*func NewReadWriter(r *Reader,w *Writer) *ReadWriter
   	*/
   	b := bytes.NewBuffer(make([]byte,0))
   	bw := bufio.NewWriter(b)
   	s := strings.NewReader("123")
   	br := bufio.NewReader(s)
   	rw := bufio.NewReadWriter(br,bw)
   	p,_ := rw.ReadString('\n')
   	fmt.Printf("%s\n",string(p))

   	rw.WriteString("hello world")
   	rw.Flush()
   	fmt.Println(b)
    
   	/*
   		NewReader读取数据存放于Reader中
   	*func NewReader(rd io.Reader) *Reader
   	*/	
   	b := bufio.NewReader(os.Stdin)
   	s,_ := b.ReadString('\n')	//读取字符直到首次出现回车
   	fmt.Println(s)
    
   	/*
   		Buffered方法返回可从缓冲区中读出数据的字节数
	*func (b *Reader) Buffered() int
	*/
	/*
		Peek返回输入流下n个字节且不会移动位置，返回的[]byte只在下一次调用读取操作前合法，若Peek返回的切片长度比n小，它也会返回一个错误说明原因，若n比缓冲尺寸大，返回的错误将是ErrBufferFull
	*func (b *Reader) Peek(n int) ([]byte,error)
	*/
    /*
    	Read从b中读出数据存放到p中，返回读出的字节数和遇到的错误
    	若缓存不为空则只能读出缓存中存在的数据，不会从底层io.Reader读取
    	若缓存为空，则：
    	1、若len(p) >= 缓存大小，直接从底层io.Reader中读出到p中
    	2、若len(p) < 缓存大小，将数据从底层读取到缓存中，再从缓存中读取到p中
    *func (b *Reader) Read(p []byte) (n int,err error)
    */
    
    /*
    	Reset方法丢弃所有的缓存数据，重置所有的状态且将缓存读切换到r
    *func (b *Reader) Reset(r io.Reader)
    */
    s := strings.NewReader("ABCDEF")
   	str := strings.NewReader("123456")
  	br := bufio.NewReader(s)
   	b,_ := br.ReadString('\n')
   	fmt.Println(b)
   	br.Reset(str)
   	b,_ = br.ReadString('\n')
   	fmt.Println(b)
    /*
    	输出：
    	ABCDEF
		123456
    */
    
    /*
    	Size方法返回缓存区的大小（以字节为单位）
    *func (b *Reader) Size() int	
    */
    s := strings.NewReader("ABCDEF")
    b := bufio.NewReader(s)
    n := b.Size()
    fmt.Println(n)
    
    /*
    	UnreadByte方法作用是撤销最近一次读出的字节，只要有内容读出就可以使用该方法撤销一个字节
    *func (b *Reader) UnreadByte() error
    	UnreadRune方法的作用是返回最近一次调用的unicode码值，若最近一次读取的不是调用的ReadRune，会返回错误
    *func (b *Reader) UnreadRune() error
    	Err方法返回扫描仪遇到的第一个非EOF错误
    *func (s *Scanner) Err() error
    */
    
    /*
    	Scan方法获取当前位置的token（该token可以通过Bytes或Text方法获取），并让Scanner的扫描位置移动到下一个token，当抵达输入流结尾或遇到错误而停止时返回false,Err方法此时返回nil
    *func (s *Scanner) Scan() bool
    */
    s := strings.NewReader("asffgghjhjhjk")
	scanner := bufio.NewScanner(s)
	flag := scanner.Scan()
	fmt.Println(flag)
    
    /*
    	split方法设置该Scanner()的分割函数，本方法必须在Scan之前调用
    *func (s *Scanner) Split(split SplitFunc)
    */
    const input = "1234 5678 1234567901234567"
	scanner := bufio.NewScanner(strings.NewReader(input))
	split := func(data []byte, atEOF bool) (advance int, token []byte, err error) {
		advance, token, err = bufio.ScanWords(data, atEOF)
		if err == nil && token != nil {
			_, err = strconv.ParseInt(string(token), 10, 32)
		}
		return
	}
	scanner.Split(split)
	for scanner.Scan() {
		fmt.Printf("%s \n", scanner.Text())
	}
    /*
    	输出：
    	1234
    	5678
    */
    
    /*
    	Text方法创建并返回最近一次scan调用生成的token
    *func (s *Scanner) Text() string
    */
    s := strings.NewReader("asvgfgghgjh\nsddgffgfg")
	scanner := bufio.NewScanner(s)
	for scanner.Scan() {
		fmt.Println(scanner.Text())
	}
    /*
    	输出：
    	asvgfgghgjh
		sddgffgfg
    */
}

```

### Builtin（为go提供一些内置的类型和函数）

```go
/*
	func append(slice []Type,elems ...Type) []Type
	func cap(v Type) int
	func close(c chan<- Type)
	func complex(r,i FloatType) ComplexType
	func copy(dst,src []Type) int
	func delete(m map[Type]Type1,key Type)
	func imag(c ComplexType) FloatType
	func len(v Type) int
	func make(t Type,size ...IntegerType) Type
	func new(Type) *Type
	func panic(v interface{})
	func print(args ...Type)
	func println(args ...Type)
	func real(c ComplexType) FloatType
	func recover() interface{}
	type ComplexType
	type FloatType
	type IntegerType
    type Type
    type Type1
    type bool
    type byte
    type complex128
    type complex64
    type error
    type float32
    type float64
    type int
    type int16
    type int32
    type int64
    type int8
    type rune
    type string
    type uint
    type uint16
    type uint32
    type uint64
    type uint8
    type uintptr
*/

```

### Bytes（主要提供操作字节切片的函数和方法）

```go
/*
	func Compare(a, b []byte) int
    func Contains(b, subslice []byte) bool
    func ContainsAny(b []byte, chars string) bool
    func ContainsRune(b []byte, r rune) bool
    func Count(s, sep []byte) int
    func Equal(a, b []byte) bool
    func EqualFold(s, t []byte) bool
    func Fields(s []byte) [][]byte
    func FieldsFunc(s []byte, f func(rune) bool) [][]byte
    func HasPrefix(s, prefix []byte) bool
    func HasSuffix(s, suffix []byte) bool
    func Index(s, sep []byte) int
    func IndexAny(s []byte, chars string) int
    func IndexByte(b []byte, c byte) int
    func IndexFunc(s []byte, f func(r rune) bool) int
    func IndexRune(s []byte, r rune) int
    func Join(s [][]byte, sep []byte) []byte
    func LastIndex(s, sep []byte) int
    func LastIndexAny(s []byte, chars string) int
    func LastIndexByte(s []byte, c byte) int
    func LastIndexFunc(s []byte, f func(r rune) bool) int
    func Map(mapping func(r rune) rune, s []byte) []byte
    func Repeat(b []byte, count int) []byte
    func Replace(s, old, new []byte, n int) []byte
    func ReplaceAll(s, old, new []byte) []byte
    func Runes(s []byte) []rune
    func Split(s, sep []byte) [][]byte
    func SplitAfter(s, sep []byte) [][]byte
    func SplitAfterN(s, sep []byte, n int) [][]byte
    func SplitN(s, sep []byte, n int) [][]byte
    func Title(s []byte) []byte
    func ToLower(s []byte) []byte
    func ToLowerSpecial(c unicode.SpecialCase, s []byte) []byte
    func ToTitle(s []byte) []byte
    func ToTitleSpecial(c unicode.SpecialCase, s []byte) []byte
    func ToUpper(s []byte) []byte
    func ToUpperSpecial(c unicode.SpecialCase, s []byte) []byte
    func ToValidUTF8(s, replacement []byte) []byte
    func Trim(s []byte, cutset string) []byte
    func TrimFunc(s []byte, f func(r rune) bool) []byte
    func TrimLeft(s []byte, cutset string) []byte
    func TrimLeftFunc(s []byte, f func(r rune) bool) []byte
    func TrimPrefix(s, prefix []byte) []byte
    func TrimRight(s []byte, cutset string) []byte
    func TrimRightFunc(s []byte, f func(r rune) bool) []byte
    func TrimSpace(s []byte) []byte
    func TrimSuffix(s, suffix []byte) []byte
type Buffer
    func NewBuffer(buf []byte) *Buffer
    func NewBufferString(s string) *Buffer
    func (b *Buffer) Bytes() []byte
    func (b *Buffer) Cap() int
    func (b *Buffer) Grow(n int)
    func (b *Buffer) Len() int
    func (b *Buffer) Next(n int) []byte
    func (b *Buffer) Read(p []byte) (n int, err error)
    func (b *Buffer) ReadByte() (byte, error)
    func (b *Buffer) ReadBytes(delim byte) (line []byte, err error)
    func (b *Buffer) ReadFrom(r io.Reader) (n int64, err error)
    func (b *Buffer) ReadRune() (r rune, size int, err error)
    func (b *Buffer) ReadString(delim byte) (line string, err error)
    func (b *Buffer) Reset()
    func (b *Buffer) String() string
    func (b *Buffer) Truncate(n int)
    func (b *Buffer) UnreadByte() error
    func (b *Buffer) UnreadRune() error
    func (b *Buffer) Write(p []byte) (n int, err error)
    func (b *Buffer) WriteByte(c byte) error
    func (b *Buffer) WriteRune(r rune) (n int, err error)
    func (b *Buffer) WriteString(s string) (n int, err error)
    func (b *Buffer) WriteTo(w io.Writer) (n int64, err error)
type Reader
    func NewReader(b []byte) *Reader
    func (r *Reader) Len() int
    func (r *Reader) Read(b []byte) (n int, err error)
    func (r *Reader) ReadAt(b []byte, off int64) (n int, err error)
    func (r *Reader) ReadByte() (byte, error)
    func (r *Reader) ReadRune() (ch rune, size int, err error)
    func (r *Reader) Reset(b []byte)
    func (r *Reader) Seek(offset int64, whence int) (int64, error)
    func (r *Reader) Size() int64
    func (r *Reader) UnreadByte() error
    func (r *Reader) UnreadRune() error
    func (r *Reader) WriteTo(w io.Writer) (n int64, err error)
*/

```

```go
func main() {
    /*
    	Compare返回一个整数表示两个[]byte切片按字典比较的结果，若a==b返回0；若a<b返回-1；若a>b返回1。nil参数视为空切片
    *func Compare(a,b []byte) int
    */
    
    /*
    	Contains判断切片b是否包含子切片subslice
    *func Contains(b,subslice []byte) bool{
    	return Index(b,subslice) != -1
    }
    */
    
    /*
    	Count计算s中有多少个不重叠的sep子切片
    *func Count(s,sep []byte) int{
    	if len(sep) == 0 {
    		return utf8.RuneCount(s) + 1
    	}
    	if len(sep) == 1 {
    		return bytealg.Count(s,sep[0])
    	}
    	n := 0
    	for {
    		i := Index(s,sep)
    		if i == -1 {
    			return n
    		}
    		n++
    		s = s[i+len(sep):]
    	}
    }
    */
    
    /*
    	判断a和b是否长度相同且包含相同字节。nil参数等效于空片
    *func Equal(a,b []byte) bool {
    	return string(a) == string(b)
    }
    */
    
    /*
    	EqualFold判断两个utf-8编码切片（将unicode大写、小写、标题三种格式字符视为相同）是否相同
    *func EqualFold(s,t []byte) bool
    */
    
    /*
    	Fields返回将字符串按照空白字符分割的多个子切片，若字符串全是空白或是空字符串的话，会返回空切片
    *func Fields(s []byte) [][]byte
    */
    fmt.Printf("Fields are: %q",bytes.Fields([]bye("  foo bar  baz   ")))
    
    /*
    	FieldsFunc使用函数f确定分割符（满足f的utf-8码值），若字符串全部是分隔符或是空字符串的话，会返回空切片
    *func FieldsFunc(s []byte,f func(rune) bool) [][]byte
    */
    f := func(c rune) bool {
        return !unicode.IsLetter(c) && !unicode.IsNumber(c)
    }
    fmt.Printf("Fields are: %q",bytes.FieldsFunc([]byte("  foo1;bar2,baz3..."),f))
    
    /*
    	HasPrefix测试字节片s是否以前缀开头，HasSuffix测试字节片s是否以后缀结尾
    *func HasPrefix(s,prefix []byte) bool {
    	return len(s) >= len(prefix) && Equal(s[0:len(prefix)],prefix)
    }
    *func HasSuffix(s,suffix []byte) bool {
    	return len(s) >= len(suffix) && Equal(s[len(s)-len(suffix):],suffix)
    }
    */
    
    /*
    	Index返回s中sep的第一个实例的索引；若s中不存在sep，则返回-1
    *func Index(s,sep []byte) int
    */
    
    /*
    	Join将s的元素连接起来创建一个新的字节片，分隔符sep放置在所得切片中的元素之间
    *func Join(s [][]byte,sep []byte) []byte {
    	//若s无元素则返回空字节片
    	if len(s) == 0 {return []byte{}}	
    	if len(s) == 1 {return append([]byte(nil),s[0]...)}
    	n := len(sep) * (len(s) - 1)
    	//计算所有元素和分隔符的字节总数
    	for _,v := range s {
    		n += len(v)
    	}
    	b := make([]byte,n)
    	bp := copy(n,s[0])	//记录连接位置
    	for _,v := range s[1:] {
    		bp += copy(b[bp:],sep)
    		bp += copy(b[bp:],v)
    	}
    	return b
    }
    */
    s := [][]byte{[]byte("foo"), []byte("bar"), []byte("baz")}
	fmt.Printf("%s", bytes.Join(s, []byte(",")))
    
    /*
    	LastIndex返回s中sep最后一个实例的索引：若s中不存在sep，则返回-1
    *func LastIndex(s,sep []byte) int
    */
    fmt.Println(bytes.Index([]byte("go gopher"), []byte("go")))
    
    /*
    	LastIndexByte返回s中c的最后一个实例的索引；若s中不存在c则返回-1
    *func LastIndexByte(s []byte,c byte) int {
    	for i := len(s) - 1; i >= 0; i-- {
    		if s[i] == c {
    			return i
    		}
    	}
    	return -1
    }
    */
    fmt.Println(bytes.LastIndexByte([]byte("go gopher"), byte('g')))
    
    /*
    	返回count个b串联形成的新的切片
    *func Repeat(b []byte,count int) {
    	if count == 0 {return []byte{}}
    	if count < 0 {
    		panic("bytes: negative Repeat count")
    	}else if len(b)*count/count != len(b) {
    		panic("bytes: Repeat count causes overflow")
    	}
    	
    	nb := make([]byte,len(b)*count)
    	bp := copy(nb,b)
    	for bp < len(nb) {
    		copy(nb[bp:],nb[:bp])
    		bp *= 2
    	}
    	return nb
    }
    */
    fmt.Printf("ba%s", bytes.Repeat([]byte("na"), 2))
    
    /*
    	Replace将返回slice的副本，其中有old的前n个非重叠实例被new取代，若old为空，则它在切片的开头和每个UTF-8序列之后匹配，最多产生k + 1个k - rune切片的替换；若n < 0，则替换次数没有限制
    	func Replace(s,old,new []byte,n int) []byte {
    		m := 0
    		if n != 0 {
    			//计算s切片中有多少个old
    			m = Count(s,old)
    		}
    		if m == 0 {
    			// Just return a copy
    			return append([]byte(nil),s...)
    		}
    		//若要替换的数小于0或大于m则全替换
    		if n < 0 || m < n {
    			n = m
    		}
            //计算缓冲区新切片总数
            t := make([]byte,len(s)+n*(len(new)-len(old)))
            w := 0
            start := 0
            for i := 0; i < n; i++ {
            	j := start
            	if len(old) == 0 {
            		if i > 0 {
            			_,wid := utf8.DecodeRune(s[start:])
            			j += wid
            		}
            	} else {
            		j += Index(s[start:],old)
            	}
            	w += copy(t[w:],s[start:j])
            	w += copy(t[w:],new)
            	start = j + len(old)
            }
            w += copy(t[w:],s[start:])
            return t[0:w]
    	}
    	
    	func ReplaceAll(s, old, new []byte) []byte {
    		return Replace(s,old,new,-1)
    	}
    */
    fmt.Printf("%s\n", bytes.Replace([]byte("oink oink oink"), []byte("k"), []byte("ky"), 2))
    
    /*
    	符文将s解释为UTF-8编码的代码点序列。它返回与s等效的一片符文(Unicode代码点)
    	func Runes(s []byte) []rune {
    		t := make([]rune,utf8.RuneCount(s))
    		i := 0
    		for len(s) > 0 {
    			r,l := utf8.DecodeRune(s)
    			t[i] = r
    			i++
    			s = s[l:]
    		} 
    		return t
    	}
    */
    rs := bytes.Runes([]byte("go gopher"))
    for _,r := range rs {
        fmt.Printf("%#U\n",r)
    }
    /*
    	输出：
    	U+0067 'g'
        U+006F 'o'
        U+0020 ' '
        U+0067 'g'
        U+006F 'o'
        U+0070 'p'
        U+0068 'h'
        U+0065 'e'
        U+0072 'r'
    */
    
    /*
    	将片段s分割为所有由sep分隔的子片段，返回这些分隔符之间的子片段的片段，若sep为空，则Split在每个UTF-8序列后拆分，它等效于SplitN，计数为-1
    	func Split(s,sep []byte) [][]byte
    */
    fmt.Printf("%q\n", bytes.Split([]byte("a,b,c"), []byte(",")))
    /*
    	输出：["a" "b" "c"]
    */
    
    /*
    	Trim通过切掉cutset中包含的所有前导和尾随UTF-8编码的代码点返回s的子片段
    	func Trim(s []byte,cutset string) []byte
    */
    fmt.Printf("[%q]", bytes.Trim([]byte(" !!! Achtung! Achtung! !!! "), "! "))
    /*
    	输出：["Achtung! Achtung"]
    */
}

```

### Context

#### Goroutine

轻量级的执行线程，多个goroutine比一个线程轻量，是go的基本执行单元；每个go程序至少有一个主goroutine（程序启动时自动创建）；使用方法类似其它语言的协程（coroutine）

```go
func Hello() {
    fmt.Println("hello everybody")
}

func main() {
    go Hello()
    fmt.Println("Golang-Gorontine Example")
}

// 此时并未执行Hello方法，main执行完后直接退出了，需要使用通道让Hello-Goroutine告诉main执行完成后退出
```

#### Channel

多个goroutine之间的沟通渠道，用于将结果、错误或任何信息从一个传递到另一个；通道是有类型的

```go
func Hello(ch chan int)  {
    fmt.Println("hello everybody")
    ch <- 1
}

func main()  {
    ch := make(chan int)
    go Hello(ch)
    <-ch
    fmt.Println("Golang-Gorontine Example")
}
```

当请求进来时，Handler创建一个监控goroutine并打印信息：

```go
func main()  {
    http.HandleFunc("/", SayHello) // 设置访问的路由
    log.Fatalln(http.ListenAndServe(":8080",nil))
}

func SayHello(writer http.ResponseWriter, request *http.Request)  {
    fmt.Println(&request)

    go func() {
        for range time.Tick(time.Second) {
            fmt.Println("Current request is in progress")
        }
    }()

    time.Sleep(2 * time.Second)
    writer.Write([]byte("Hi, New Request Comes"))
}
```

此时假定请求耗时2s，在请求2s后返回，期望打印2次后立即停止，但运行发现，监控goroutine打印2次后仍不会结束且会一直打印；问题在创建监控goroutine后未对生命周期作控制；下面在打印前检测`request.Context()`是否已经结束，若结束则退出循环

```go
func main()  {
    http.HandleFunc("/", SayHello) // 设置访问的路由

    log.Fatalln(http.ListenAndServe(":8080",nil))
}

func SayHello(writer http.ResponseWriter, request *http.Request)  {
    fmt.Println(&request)

    go func() {
        for range time.Tick(time.Second) {
            select {
            case <- request.Context().Done():
                fmt.Println("request is outgoing")
                return
            default:
                fmt.Println("Current request is in progress")
            }
        }
    }()

    time.Sleep(2 * time.Second)
    writer.Write([]byte("Hi, New Request Comes"))
}
```

基于如上需求，context包应运而生；**context可以提供一个请求，从API请求边界到各goroutine请求域数据传递、取消信号及截止时间等能力**

#### 设计原理

Go中每一个请求都是通过一个单独的Goroutine进行处理的，HTTP/RPC请求处理器往往会启动新的Goroutine访问数据库和RPC服务，可能会创建多个来处理一次请求；Context的主要作用即在不同Goroutine间同步请求特定的数据、取消信号以及处理请求的截止日期

#### Context接口

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <- chan struct{}
    Err() error
    Value(key interface{}) interface{}
}

// Deadline方法需要返回当前Context被取消的时间，即完成工作的截止时间
// Done方法返回一个Channel，该Channel会在当前工作完成或上下文被取消后关闭，多次调用该方法会返回同一个Channel
// Err方法返回Context结束的原因，只会在Done返回的Channel被关闭时才会返回非空的值
	// 若当前Context被取消就会返回Canceled错误
	// 若当前Context超时就会返回DeadlineExceed错误
// Value方法从Context中返回键对应的值对于同一个上下文来说，多次调用Value 并传入相同的Key会返回相同的结果，该方法仅用于传递跨API和进程间跟请求域数据

// 跨域：若a页面想获取b页面资源，但它们的协议、域名、端口、子域名不同；则所进行的访问行动都是跨域的（跨域限制访问其实是浏览器的限制）
// 跨域问题解决：用nginx作为代理服务器只在80端口交互

// Go内置两个函数：Background()和TODO()，这两个函数分别返回一个实现了Context接口的background和todo;
// Background()主要用于main函数、初始化以及测试代码中，作为Context这个树结构的最顶层的Context，也就是根Context
// TODO()，它目前还不知道具体的使用场景，如果我们不知道该使用什么Context的时候，可以使用这个
// background和todo本质上都是emptyCtx结构体类型，是一个不可取消，没有设置截止时间，没有携带任何值的Context
```

##### With系列函数

此外，`context`包中还定义了四个With系列函数；这些函数返回的都是实现了上述接口方法的对应结构体

![](https://s1.ax1x.com/2023/02/15/pS7P1MD.png)

###### WithCancel

接受一个Context并返回其子Context和取消函数cancel

新创建协程中传入子Context做参数且监控其Done通道，若收到消息则退出

需要新协程结束时在外调用cancel函数，即会往子Context的Done通道发送消息

当父Context的Done()关闭时，子ctx的Done()也会被关闭

```go
// 利用根context创建一个父context，并使用其创建一个协程
// 再由父创建一个子context，再使用该子context创建一个协程
// 一段时间后调用父context的cancel函数发现父和子协程都收到信号结束了
package main
 
import (
	"context"
	"fmt"
	"time"
)
 
func main() {
	// 父context(利用根context得到)
	ctx, cancel := context.WithCancel(context.Background())
	
    // 父context的子协程
	go watch1(ctx)
 
	// 子context，注意：这里虽然也返回了cancel的函数对象，但是未使用
	valueCtx, _ := context.WithCancel(ctx)
	// 子context的子协程
	go watch2(valueCtx)
 
	fmt.Println("现在开始等待3秒,time=", time.Now().Unix())
	time.Sleep(3 * time.Second)
 
	// 调用cancel()
	fmt.Println("等待3秒结束,调用cancel()函数")
	cancel()
 
	// 再等待5秒看输出，可以发现父context的子协程和子context的子协程都会被结束掉
	time.Sleep(5 * time.Second)
	fmt.Println("最终结束,time=", time.Now().Unix())
}
 
// 父context的协程
func watch1(ctx context.Context) {
	for {
		select {
		case <-ctx.Done(): //取出值即说明是结束信号
			fmt.Println("收到信号，父context的协程退出,time=", time.Now().Unix())
			return
		default:
			fmt.Println("父context的协程监控中,time=", time.Now().Unix())
			time.Sleep(1 * time.Second)
		}
	}
}
 
// 子context的协程
func watch2(ctx context.Context) {
	for {
		select {
		case <-ctx.Done(): //取出值即说明是结束信号
			fmt.Println("收到信号，子context的协程退出,time=", time.Now().Unix())
			return
		default:
			fmt.Println("子context的协程监控中,time=", time.Now().Unix())
			time.Sleep(1 * time.Second)
		}
	}
}
```

```go
// withCancel的实现
func WithCancel(parent Context) (ctx Context, cancel CancelFunc) {
	if parent == nil {
		panic("cannot create context from nil parent")
	}
	c := newCancelCtx(parent)   //返回一个cancelCtx结构体
	propagateCancel(parent, &c)   //绑定父子context关系
	return &c, func() { c.cancel(true, Canceled) }
}

type cancelCtx struct {
	Context   						// 自己的父context

	mu       sync.Mutex				
	done     chan struct{}         // 接收取消信号的管道
	children map[canceler]struct{} // 包含自己路径下所有子context集合
	err      error
}

//ctx.Done()其实就是一个管道，即上面cancelCtx结构体中的done
func (c *cancelCtx) Done() <-chan struct{} {
	c.mu.Lock()
	if c.done == nil {
		c.done = make(chan struct{})
	}
	d := c.done
	c.mu.Unlock()
	return d
}
```

###### WithTimeout、WithDeadline

`WithTimeout`底层实现是调用的`WithDeadline`，并添加对应的过期时间

`WithDeadline`即添加了`timer`计时器，`time.AfterFunc`中计时条件满足时自动触发`cancel`函数

###### WithValue

返回一个带有键值对的`valueCtx`结构体：

```go
type valueCtx struct {
	Context
	key, val interface{}
}
```

### Database

```go
type DB struct{}	//数据库操作句柄，代表一个具有零到多个底层连接的连接池，可以安全的被多个go程同时使用；sql包会自动创建和释放连接，也会维护一个闲置连接的连接池；连接池的大小可用SetMaxldleConns方法控制

func Open(driverName,dataSourceName string) (*DB,error)
/*
	driverName表示driver名称，dataSourceName表示连接数据库的信息；
	Open函数只验证其参数，不创建与数据库的连接，若要检查数据源的名称是否合法，应调用返回值的Ping方法；Open函数只需调用一次，很少需要关闭DB
*/

func (db *DB) Ping() error		
// 检查与数据库的连接是否有效

func (db *DB) Close() error		
// 关闭数据库，释放任何打开的资源，一般不关闭DB，DB句柄通常被多个go程共享，并长期活跃

func (db *DB) Exec(query string, args ...interface{}) (Result,error)
// Exec执行一次命令（包括查询、删除、更新、插入等），不返回任何执行结果

func (*DB) Query(query string,args ...interface{}) (*Rows,error)
// Query执行一次查询，返回多行结果，一般用于执行select命令
    age := 27
    rows, err := db.Query("SELECT name FROM users WHERE age=?", age)
    if err != nil {
        log.Fatal(err)
    }
    defer rows.Close()
    for rows.Next() {
        var name string
        if err := rows.Scan(&name); err != nil {
            log.Fatal(err)
        }
        fmt.Printf("%s is %d\n", name, age)
    }
    if err := rows.Err(); err != nil {
        log.Fatal(err)
    }

func (db *DB) QueryRow(query string,args ...interface{}) *Row
// QueryRow执行一次查询并期望返回最多一行结果；QueryRow总是返回非nil的值，直到返回值的Scan方法被调用时才会返回被延迟的错误
    id := 123
    var username string
    err := db.QueryRow("SELECT username FROM users WHERE id=?", id).Scan(&username)
    switch {
    case err == sql.ErrNoRows:
        log.Printf("No user with that ID.")
    case err != nil:
        log.Fatal(err)
    default:
        fmt.Printf("Username is %s\n", username)
    }

func (*DB) Begin() (*Tx,error)
// 开始一个事务

func (rs *Rows) Next() bool
// 用于Scan方法的下一行结果，若成功返回真，若没有下一行或出现错误返回假
```

### encoding

#### json

`struct tag`可以决定`Marshal`和`Unmarshal`函数如何序列化和反序列化数据

##### Marshal

`func Marshal(v interface{}) ([]byte, error)`

```go
type ColorGroup struct {
    ID     int
    Name   string
    Colors []string
}
group := ColorGroup{
    ID:     1,
    Name:   "Reds",
    Colors: []string{"Crimson", "Red", "Ruby", "Maroon"},
}
b, err := json.Marshal(group)
if err != nil {
    fmt.Println("error:", err)
}
os.Stdout.Write(b)

// Output: {"ID":1,"Name":"Reds","Colors":["Crimson","Red","Ruby","Maroon"]}
```

##### Unmarshal

`func Unmarshal(data []byte, v interface{}) error`

```go
var jsonBlob = []byte(`[
	{"Name": "Platypus", "Order": "Monotremata"},
	{"Name": "Quoll",    "Order": "Dasyuromorphia"}
]`)
type Animal struct {
    Name  string
    Order string
}
var animals []Animal
err := json.Unmarshal(jsonBlob, &animals)
if err != nil {
    fmt.Println("error:", err)
}
fmt.Printf("%+v", animals)

// Output: [{Name:Platypus Order:Monotremata} {Name:Quoll Order:Dasyuromorphia}]
```

### fmt

实现了类似C语言`printf`和`scanf`的格式化I/O

#### 类型格式

##### 通用

```
%v		值的默认格式表示
%+v		类似%v，但输出结构体时会添加字段名
%#v		值的go语法表示
%T		值的类型的go语法表示
%%		百分号
```

##### 布尔值

```
%t		单词true或false
```

##### 整数

```
%b		表示为二进制
%c		该值对应的unicode码值
%d		表示为十进制
%o		表示为八进制
%q		该值对应的单引号括起来的go语法字符字面值，必要时采用安全的转义表示
%x		表示为十六进制，使用a-f
%X		表示为十六进制，使用A-F
%U		表示为Unicode格式：U+1234，等价于"U+%04X"
```

##### 浮点数与复数

```
%b		无小数部分、二进制指数的科学计数法，如-123456p-78;
%e		科学计数法，如-1234.456e+78
%E		科学计数法，如-1234.456E+78
%f		有小数部分但无指数部分，如123.456（在后面加入小数，个位表示宽度，小数位表示精度）
%F		等价于%f
%g		根据实际情况采用%e或%f格式（以获得更简洁、准确的输出）
%G		根据实际情况采用%E或%F格式（以获得更简洁、准确的输出）
```

##### 字符串与[]byte

```
%s		直接输出字符串或者[]byte
%q		该值对应双引号括起来的go语法字符串字面值，必要时采用安全的转义表示
%x		每个字节用两字符十六进制数表示（使用a-f）
%X		每个字节用两字符十六进制数表示（使用A-F）    
```

##### 指针

```
%p		表示为十六进制，并加上前导的0x    
fmt.Sprintf("%[2]d %[1]d\n", 11, 22)	//会生成"22 11"
```

#### 向外输出

##### Print

```go
//直接输出内容
func Print(a ...interface{}) (n int,err error)
//支持格式化输出字符串
func Printf(format string,a ...interface{}) (n int,err error)
//在输出内容结尾添加一个换行符
func Println(a ...interface{}) (n int,err error)

fmt.Printf("名字是：%v\n","lxx")
fmt.Printf("年龄是：%v\n",19)
p:= struct {
	name string
	age int
}{"lxx",19}
fmt.Printf("结构体内容为：%v\n",p)
fmt.Printf("结构体内容为(带字段名)：%+v\n",p) // 输出结构体是会带name
fmt.Printf("结构体内容为(值的Go语法表示)：%#v\n",p) // 输出结构体是会带name

fmt.Printf("切片内容为：%v\n",[]int{4,5,6})
fmt.Printf("切片内容为(值的Go语法表示)：%#v\n",[]int{4,5,6})

fmt.Printf("切片值的类型为：%T\n",[]int{4,5,6})
fmt.Printf("字符串值的类型为：%T\n","lxx")

fmt.Printf("打印百分百：100%%\n")

//名字是：lxx
//年龄是：19
//结构体内容为：{lxx 19}
//结构体内容为(带字段名)：{name:lxx age:19}
//结构体内容为(值的Go语法表示)：struct { name string; age int }{name:"lxx", age:19}
//切片内容为：[4 5 6]
//切片内容为(值的Go语法表示)：[]int{4, 5, 6}
//切片值的类型为：[]int
//字符串值的类型为：string
//打印百分百：100%
```

##### Fprint

将内容输出到一个`io.Writer`接口类型的变量`w`中，通常用该函数往文件中写入内容

```go
func Fprint(w io.Writer, a ...interface{}) (n int, err error)
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error)
func Fprintln(w io.Writer, a ...interface{}) (n int, err error)

func main() {
    //向标准输出写入内容
    fmt.Fprint(os.Stdout,"向标准输出（控制台）写入内容")
    fileObj,err := os.OpenFile("./xx.txt",os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
    if err != nil {
		fmt.Println("打开文件出错，err:", err)
		return
	}
    name := "lxx is nb"
    // 向打开的文件句柄中写入内容
	fmt.Fprintf(fileObj, "往文件中(标准输出)写如信息：%s", name)
}
```

##### Sprint

将传入的数据生成并返回一个字符串

```go
func Sprint(a ...interface{}) string
func Sprintf(format string, a ...interface{}) string
func Sprintln(a ...interface{}) string

s1 := fmt.Sprint("lxx")
name := "lxx"
age := 18
s2 := fmt.Sprintf("姓名:%s,年龄:%d", name, age)
s3 := fmt.Sprintln("lxx is nb")
fmt.Println(s1, s2, s3)

//Output: lxx 姓名:lxx,年龄:18 lxx is nb
```

#### 获取输入

##### Scan

- 从标准输入扫描文本，读取由空白字符分隔的值保存到传递给本函数的参数中，换行符视为空白符
- 返回成功扫描的数据个数和遇到的任何错误，若读取数据个数比提供的参数少，会返回一个错误报告原因
- **若想完整获取输入的内容，而输入的内容可能包含空格，可以使用`bufio`中的方法**

`func Scan(a ...interface{}) (n int,err error)`

##### Scanf

`func Scanf(format string, a ...interface{}) (n int, err error)`

```go
var (
	name string
    age int
)
fmt.Scanf("name:%s age:%d", &name, &age) // 在控制台按照该格式输入
fmt.Printf("扫描结果： 姓名:%s 年龄:%d \n", name, age)

// 控制台按如下格式输入
//name:lxx age:19
//扫描结果： 姓名:lxx 年龄:19 
```

##### Scanln

类似Scan，遇到换行时才停止扫描

`func Scanln(a ...interface{}) (n int, err error)`

##### Fscan和Sscan

Fscan从`io.Reader`中读取数据，Sscan从指定字符串中读取数据

```go
func Fscan(r io.Reader, a ...interface{}) (n int, err error)
func Fscanln(r io.Reader, a ...interface{}) (n int, err error)
func Fscanf(r io.Reader, format string, a ...interface{}) (n int, err error)

func Sscan(str string, a ...interface{}) (n int, err error)
func Sscanln(str string, a ...interface{}) (n int, err error)
func Sscanf(str string, format string, a ...interface{}) (n int, err error)

var name string="lxx"
var newName string=""
fmt.Sscan(name,&newName) // 相当于把name的值赋值给newName
fmt.Println(name)
fmt.Println(newName)
```

### strings 和 strconv

对于字符串的预定义处理函数

#### 前缀和后缀

`HasPrefix()`判断字符串`s`是否以`prefix`开头

`HasSuffix()` 判断字符串 `s` 是否以 `suffix` 结尾

```go
strings.HasPrefix(s, prefix string) bool
strings.HasSuffix(s, suffix string) bool
```

#### 字符串包含关系

`Contains()` 判断字符串 `s` 是否包含 `substr`

```go
strings.Contains(s, substr string) bool
```

`Index()` 返回字符串 `str` 在字符串 `s` 中的索引（`str` 的第一个字符的索引），`-1` 表示字符串 `s` 不包含字符串 `str`

```go
strings.Index(s, str string) int
```

`LastIndex()` 返回字符串 `str` 在字符串 `s` 中最后出现位置的索引（`str` 的第一个字符的索引），`-1` 表示字符串 `s` 不包含字符串 `str`

```go
strings.LastIndex(s, str string) int
```

若要查询非 ASCII 编码的字符在父字符串中的位置，建议使用以下函数定位

```go
strings.IndexRune(s string, r rune) int
```

#### 字符串替换

`Replace()` 用于将字符串 `str` 中的前 `n` 个字符串 `old` 替换为字符串 `new`，并返回一个新的字符串，如果 `n = -1` 则替换所有字符串 `old` 为字符串 `new`

```go
strings.Replace(str, old, new string, n int) string
```

#### 字符串出现次数

`Count()` 用于计算字符串 `str` 在字符串 `s` 中出现的非重叠次数

```go
strings.Count(s, str string) int
```

#### 重复字符串

`Repeat()` 用于重复 `count` 次字符串 `s` 并返回一个新的字符串

```go
strings.Repeat(s, count int) string
```

#### 修改字符串大小写

`ToLower()` 将字符串中的 Unicode 字符全部转换为相应的小写字符

`ToUpper()` 将字符串中的 Unicode 字符全部转换为相应的大写字符

```go
strings.ToLower(s) string
strings.ToUpper(s) string
```

#### 修剪字符串

使用 `strings.TrimSpace(s)` 来剔除字符串开头和结尾的空白符号；若想剔除指定字符，可以使用 `strings.Trim(s, "cut")` 来将开头和结尾的 `cut` 去除掉。该函数的第二个参数可以包含任何字符，若只想剔除开头或者结尾的字符串，则可以使用 `TrimLeft()` 或者 `TrimRight()` 来实现

#### 分割字符串

`strings.Fields(s)` 将会利用 1 个或多个空白符号来作为动态长度的分隔符将字符串分割成若干小块，并返回一个 slice，如果字符串只包含空白符号，则返回一个长度为 0 的 slice

`strings.Split(s, sep)` 用于自定义分割符号进行分割，同样返回 slice

因为这 2 个函数都会返回 slice，所以习惯使用 for-range 循环来对其进行处理

#### 拼接slice到字符串

`Join()` 用于将元素类型为 string 的 slice 用分割符号来拼接组成一个字符串

```go
strings.Join(sl []string, sep string) string
```

#### 从字符串中读取内容

 `strings.NewReader(str)` 用于生成一个 `Reader` 并读取字符串中的内容，然后返回指向该 `Reader` 的指针，从其它类型读取内容的函数还有：

- `Read()` 从 `[]byte` 中读取内容
- `ReadByte()` 和 `ReadRune()` 从字符串中读取下一个 `byte` 或者 `rune`

#### 字符串与其它类型的转换

与字符串相关的类型转换都是通过 `strconv` 包实现的，任何类型`T`转换为字符串总是成功的

针对从数字类型转换到字符串，Go 提供了以下函数：

- `strconv.Itoa(i int) string` 返回数字 `i` 表示的字符串类型的十进制数
- `strconv.FormatFloat(f float64, fmt byte, prec int, bitSize int) string`将 64 位浮点型的数字转换为字符串，其中 `fmt` 表示格式（其值可以是 `'b'`、`'e'`、`'f'` 或 `'g'`），`prec` 表示精度，`bitSize` 则使用 32 表示 `float32`，用 64 表示 `float64`

针对从字符串类型转换为数字类型，Go 提供了以下函数：

- `strconv.Atoi(s string) (i int, err error)` 将字符串转换为 `int` 型
- `strconv.ParseFloat(s string, bitSize int) (f float64, err error)` 将字符串转换为 `float64` 型

### IO

```go
/*
Constants
Variables
func Copy(dst Writer, src Reader) (written int64, err error)
func CopyBuffer(dst Writer, src Reader, buf []byte) (written int64, err error)
func CopyN(dst Writer, src Reader, n int64) (written int64, err error)
func Pipe() (*PipeReader, *PipeWriter)
func ReadAll(r Reader) ([]byte, error)
func ReadAtLeast(r Reader, buf []byte, min int) (n int, err error)
func ReadFull(r Reader, buf []byte) (n int, err error)
func WriteString(w Writer, s string) (n int, err error)
type ByteReader
type ByteScanner
type ByteWriter
type Closer
type LimitedReader
	func (l *LimitedReader) Read(p []byte) (n int, err error)
type PipeReader
	func (r *PipeReader) Close() error
	func (r *PipeReader) CloseWithError(err error) error
	func (r *PipeReader) Read(data []byte) (n int, err error)
type PipeWriter
	func (w *PipeWriter) Close() error
	func (w *PipeWriter) CloseWithError(err error) error
	func (w *PipeWriter) Write(data []byte) (n int, err error)
type ReadCloser
func NopCloser(r Reader) ReadCloser
type ReadSeekCloser
type ReadSeeker
type ReadWriteCloser
type ReadWriteSeeker
type ReadWriter
type Reader
	func LimitReader(r Reader, n int64) Reader
	func MultiReader(readers ...Reader) Reader
	func TeeReader(r Reader, w Writer) Reader
type ReaderAt
type ReaderFrom
type RuneReader
type RuneScanner
type SectionReader
	func NewSectionReader(r ReaderAt, off int64, n int64) *SectionReader
	func (s *SectionReader) Read(p []byte) (n int, err error)
	func (s *SectionReader) ReadAt(p []byte, off int64) (n int, err error)
	func (s *SectionReader) Seek(offset int64, whence int) (int64, error)
	func (s *SectionReader) Size() int64
type Seeker
type StringWriter
type WriteCloser
type WriteSeeker
type Writer
	func MultiWriter(writers ...Writer) Writer
type WriterAt
type WriterTo
*/
```

```go
func main() {
    /*
    *func Copy(dst Writer,src Reader) (written int64,err error)
    */
    r := strings.NewReader("some io.Reader stream to be read")
    if _, err := io.Copy(os.Stdout, r); err != nil {
		log.Fatal(err)
	}
    
    /*
    *func Pipe() (*PipeReader,*PipeWriter)
    */
    r,w := io.Pipe()
    go func() {
        fmt.Fprint(w,"some io.Reader stream to be read\n")
        w.Close()
    }()
    if _,err := io.Copy(os.Stdout,r); err != nil {
        log.Fatal(err)
    }
}
```

### log（提供日志操作的函数和方法）

```go
/*
Constants
func Fatal(v ...interface{})
func Fatalf(format string, v ...interface{})
func Fatalln(v ...interface{})
func Flags() int
func Output(calldepth int, s string) error
func Panic(v ...interface{})
func Panicf(format string, v ...interface{})
func Panicln(v ...interface{})
func Prefix() string
func Print(v ...interface{})
func Printf(format string, v ...interface{})
func Println(v ...interface{})
func SetFlags(flag int)
func SetOutput(w io.Writer)
func SetPrefix(prefix string)
func Writer() io.Writer
type Logger
	func Default() *Logger
	func New(out io.Writer, prefix string, flag int) *Logger
	func (l *Logger) Fatal(v ...interface{})
	func (l *Logger) Fatalf(format string, v ...interface{})
	func (l *Logger) Fatalln(v ...interface{})
	func (l *Logger) Flags() int
	func (l *Logger) Output(calldepth int, s string) error
	func (l *Logger) Panic(v ...interface{})
	func (l *Logger) Panicf(format string, v ...interface{})
	func (l *Logger) Panicln(v ...interface{})
	func (l *Logger) Prefix() string
	func (l *Logger) Print(v ...interface{})
	func (l *Logger) Printf(format string, v ...interface{})
	func (l *Logger) Println(v ...interface{})
	func (l *Logger) SetFlags(flag int)
	func (l *Logger) SetOutput(w io.Writer)
	func (l *Logger) SetPrefix(prefix string)
	func (l *Logger) Writer() io.Writer
*/
```

```go
func main() {
    // Print类：跟fmt差不多，只是前面加了格式
    // Fatal类：日志输出后，系统调用os.exit(1)，整个程序退出，若后面有defer，也不执行
    // Panic类：日志输出后，发生Panic
    
    /*
    	Fatal等价于{Print(v...);os.Exit(1)}
    *func Fatal(v ...interface{})
    */
    log.Fatal("hello啊树先生，执行后终止")
    fmt.Println("你执行了吗？")
    
    /*
    	SetFlags设置标准logger的输出选项
    *func SetFlags(flag int)
    */
    log.SetFlags(log.Ldate|log.Ltime|log.LUTC)
    log.Println("sddfgfgh")
    
    /*
    	SetOutput设置标准logger的输出目的地，默认是标准错误输出
    */
    file,_ := os.Create("sdk/testLog")
    log.SetOutput(file)
}
```

### sort（按照一定规则对元素进行相关排序）

```go
/*
func Float64s(x []float64)
func Float64sAreSorted(x []float64) bool
func Ints(x []int)
func IntsAreSorted(x []int) bool
func IsSorted(data Interface) bool
func Search(n int, f func(int) bool) int
func SearchFloat64s(a []float64, x float64) int
func SearchInts(a []int, x int) int
func SearchStrings(a []string, x string) int
func Slice(x interface{}, less func(i, j int) bool)
func SliceIsSorted(x interface{}, less func(i, j int) bool) bool
func SliceStable(x interface{}, less func(i, j int) bool)
func Sort(data Interface)
func Stable(data Interface)
func Strings(x []string)
func StringsAreSorted(x []string) bool
type Float64Slice
	func (x Float64Slice) Len() int
	func (x Float64Slice) Less(i, j int) bool
	func (p Float64Slice) Search(x float64) int
	func (x Float64Slice) Sort()
	func (x Float64Slice) Swap(i, j int)
type IntSlice
	func (x IntSlice) Len() int
	func (x IntSlice) Less(i, j int) bool
	func (p IntSlice) Search(x int) int
	func (x IntSlice) Sort()
	func (x IntSlice) Swap(i, j int)
type Interface
	func Reverse(data Interface) Interface
type StringSlice
	func (x StringSlice) Len() int
	func (x StringSlice) Less(i, j int) bool
	func (p StringSlice) Search(x string) int
	func (x StringSlice) Sort()
	func (x StringSlice) Swap(i, j int)
*/
```

```go
func main() {
    /*
    	Float64s按递增顺序对float64s的一部分进行排序，Not-a-number(NaN)值先于其他值排序
    *func Float64s(x []float64)
    */
    s := []float64{math.Inf(1), math.NaN(), math.Inf(-1), 0.0}
	sort.Float64s(s)
	fmt.Println(s)
    /*输出：[NaN -Inf 0 +Inf]*/
    
    /*
    	Float64sAreSorted报告切片x是否以递增的顺序排序，其中not-a-number(NaN)值位于任何其他值之前
    *func Float64sAreSorted(x []float64) bool
    */
    
    /*
    	Search使用二分法进行查找
    *func Search(n int,f func(int) bool) int
    */
    a := []int{1,3,6,10,15,21,28,36,45,55}
    x := 6
    
    i := sort.Search(len(a),func(i int) bool {return a[i] >= x})
    if i < len(a) && a[i] == x {
        fmt.Printf("found %d at index %d in %v\n", x, i, a)
    } else {
        fmt.Printf("%d not found in %v\n", x, a)
    }
    
    /*
    	返回x在a中应该存在的位置，无论a中是否存在a中在递增顺序的a中搜索x，返回x的索引。如果查找不到，返回值是x应该插入a的位置（以保证a的递增顺序），返回值可以是len(a)
    *func SearchInts(a []int,x int) int	{
    	return Search(len(a),func(i int) bool {return a[i]>=x})
    }
    */
    s := []int{5, 2, 6, 3, 1, 4}
	index := sort.SearchInts(s, 5)
	fmt.Println(index)
    
    /*
    	字符串按升序对字符串的一部分进行排序
    *func Strings(x []string)
    */
    s := []string{"Go", "Bravo", "Gopher", "Alpha", "Grin", "Delta"}
	sort.Strings(s)
	fmt.Println(s)
    
    /*
    	求FloatSlice的数据类型的长度
    *func (x Float64Slice) Len() int
    */
    var x sort.Float64Slice
	x = []float64{74.3, 59.0, math.Inf(1), 238.2, -784.0, 2.3, math.NaN(), math.NaN(), 
        math.Inf(-1), 9845.768, -959.7485, 905, 7.8, 7.8}
	fmt.Println(x.Len())
}
```

### OS（底层相关的函数和方法）

```go
/*
Constants
Variables
func Chdir(dir string) error
func Chmod(name string, mode FileMode) error
func Chown(name string, uid, gid int) error
func Chtimes(name string, atime time.Time, mtime time.Time) error
func Clearenv()
func DirFS(dir string) fs.FS
func Environ() []string
func Executable() (string, error)
func Exit(code int)
func Expand(s string, mapping func(string) string) string
func ExpandEnv(s string) string
func Getegid() int
func Getenv(key string) string
func Geteuid() int
func Getgid() int
func Getgroups() ([]int, error)
func Getpagesize() int
func Getpid() int
func Getppid() int
func Getuid() int
func Getwd() (dir string, err error)
func Hostname() (name string, err error)
func IsExist(err error) bool
func IsNotExist(err error) bool
func IsPathSeparator(c uint8) bool
func IsPermission(err error) bool
func IsTimeout(err error) bool
func Lchown(name string, uid, gid int) error
func Link(oldname, newname string) error
func LookupEnv(key string) (string, bool)
func Mkdir(name string, perm FileMode) error
func MkdirAll(path string, perm FileMode) error
func MkdirTemp(dir, pattern string) (string, error)
func NewSyscallError(syscall string, err error) error
func Pipe() (r *File, w *File, err error)
func ReadFile(name string) ([]byte, error)
func Readlink(name string) (string, error)
func Remove(name string) error
func RemoveAll(path string) error
func Rename(oldpath, newpath string) error
func SameFile(fi1, fi2 FileInfo) bool
func Setenv(key, value string) error
func Symlink(oldname, newname string) error
func TempDir() string
func Truncate(name string, size int64) error
func Unsetenv(key string) error
func UserCacheDir() (string, error)
func UserConfigDir() (string, error)
func UserHomeDir() (string, error)
func WriteFile(name string, data []byte, perm FileMode) error
type DirEntry
	func ReadDir(name string) ([]DirEntry, error)
type File
	func Create(name string) (*File, error)
	func CreateTemp(dir, pattern string) (*File, error)
	func NewFile(fd uintptr, name string) *File
	func Open(name string) (*File, error)
	func OpenFile(name string, flag int, perm FileMode) (*File, error)
	func (f *File) Chdir() error
	func (f *File) Chmod(mode FileMode) error
	func (f *File) Chown(uid, gid int) error
	func (f *File) Close() error
	func (f *File) Fd() uintptr
	func (f *File) Name() string
	func (f *File) Read(b []byte) (n int, err error)
	func (f *File) ReadAt(b []byte, off int64) (n int, err error)
	func (f *File) ReadDir(n int) ([]DirEntry, error)
	func (f *File) ReadFrom(r io.Reader) (n int64, err error)
	func (f *File) Readdir(n int) ([]FileInfo, error)
	func (f *File) Readdirnames(n int) (names []string, err error)
	func (f *File) Seek(offset int64, whence int) (ret int64, err error)
	func (f *File) SetDeadline(t time.Time) error
	func (f *File) SetReadDeadline(t time.Time) error
	func (f *File) SetWriteDeadline(t time.Time) error
	func (f *File) Stat() (FileInfo, error)
	func (f *File) Sync() error
	func (f *File) SyscallConn() (syscall.RawConn, error)
	func (f *File) Truncate(size int64) error
	func (f *File) Write(b []byte) (n int, err error)
	func (f *File) WriteAt(b []byte, off int64) (n int, err error)
	func (f *File) WriteString(s string) (n int, err error)
type FileInfo
	func Lstat(name string) (FileInfo, error)
	func Stat(name string) (FileInfo, error)
type FileMode
type LinkError
	func (e *LinkError) Error() string
	func (e *LinkError) Unwrap() error
type PathError
type ProcAttr
type Process
	func FindProcess(pid int) (*Process, error)
	func StartProcess(name string, argv []string, attr *ProcAttr) (*Process, error)
	func (p *Process) Kill() error
	func (p *Process) Release() error
	func (p *Process) Signal(sig Signal) error
	func (p *Process) Wait() (*ProcessState, error)
type ProcessState
	func (p *ProcessState) ExitCode() int
	func (p *ProcessState) Exited() bool
	func (p *ProcessState) Pid() int
	func (p *ProcessState) String() string
	func (p *ProcessState) Success() bool
	func (p *ProcessState) Sys() interface{}
	func (p *ProcessState) SysUsage() interface{}
	func (p *ProcessState) SystemTime() time.Duration
	func (p *ProcessState) UserTime() time.Duration
type Signal
type SyscallError
	func (e *SyscallError) Error() string
	func (e *SyscallError) Timeout() bool
	func (e *SyscallError) Unwrap() error
*/
```

```go
func main() {
    /*
    	Chdir将当前工作目录更改为指定的目录，若有错，将是*PathError类型
    *func Chdir(dir string) error
    */
    beforeDir,_ := os.Getwd()
    fmt.Println(beforeDir)
    err := os.Chdir("sdk/test")
    fmt.Println(err)
    latestDir,_ := os.Getwd()
    fmt.Println(latestDir)
    
    /*
    	Chmod更改文件模式，若文件是符号链接将更改链接目标的模式
    *func Chmod(name string,mode FileMode) error
    */
    if err := os.Chmod("sdk/test.txt",0644); err != nil {
        log.Fatal(err)
    }
    
    /*
    	Chown 更改文件的uid（用户唯一标识符）和gid（用户组唯一标识符），若文件是符号链接则会更改链接目标
    */
    if err := os.Lchown(FilePath, 501, 20); err != nil {
		log.Fatal(err)
	}
    
    /*
    	Chtimes 更改指定文件的访问和修改时间，类似于Unix utime()或utimes()函数
    *func Chtimes(name string, atime time.Time, mtime time.Time) error
    */
    mtime := time.Date(2006,time.February,1,3,4,5,0,time.UTC)
    atime := time.Date(2007,time.March,2,4,5,6,0,time.UTC)
    if err := os.Chtimes("some-filename", atime, mtime); err != nil {
		log.Fatal(err)
	}
    
    /*
    	Clearenv 删除所有环境变量
    *func Clearenv()
        DirFS返回文件系统操作对象
    *func DirFS(dir string) fs.FS
    	Executable可执行文件返回启动当前进程的可执行文件的路径名称
    *func Executable() (string,error)
    */
    
    /**
	*Expand根据映射函数替换字符串中的${var}或$var。例如，os.ExpandEnv(s)
	*等效于os.Expand(s，os.Getenv)。
	*func Expand(s string, mapping func(string) string) string
	*/
	mapper := func(placeholderName string) string {
		switch placeholderName {
		case "DAY_PART":
			return "morning"
		case "NAME":
			return "Gopher"
		}
		return ""
	}
	fmt.Println(os.Expand("Good ${DAY_PART}, $NAME!", mapper))
    
    /*
    	ExpandEnv根据当前环境变量的值替换字符串中的${var}或$var，未定义变量的引用将替换为空字符串
    *func ExpandEnv(s string) string
    */
    os.Setenv("NAME", "gopher")
	os.Setenv("BURROW", "/usr/gopher")
	fmt.Println(os.ExpandEnv("$NAME lives in ${BURROW}."))
    
    /*
    	Getegid 返回调用者的数字有效组ID
    *func Getegid() int
    */
	
    /*
    	Hostname返回内核提供的主机名
    	func Hostname() (name string,err error)
    */
}
```

### time

```go
/*
Constants
func After(d Duration) <-chan Time
func Sleep(d Duration)
func Tick(d Duration) <-chan Time
type Duration
	func ParseDuration(s string) (Duration, error)
	func Since(t Time) Duration
	func Until(t Time) Duration
	func (d Duration) Hours() float64
	func (d Duration) Microseconds() int64
	func (d Duration) Milliseconds() int64
	func (d Duration) Minutes() float64
	func (d Duration) Nanoseconds() int64
	func (d Duration) Round(m Duration) Duration
	func (d Duration) Seconds() float64
	func (d Duration) String() string
	func (d Duration) Truncate(m Duration) Duration
type Location
	func FixedZone(name string, offset int) *Location
	func LoadLocation(name string) (*Location, error)
	func LoadLocationFromTZData(name string, data []byte) (*Location, error)
	func (l *Location) String() string
type Month
	func (m Month) String() string
type ParseError
	func (e *ParseError) Error() string
type Ticker
	func NewTicker(d Duration) *Ticker
	func (t *Ticker) Reset(d Duration)
	func (t *Ticker) Stop()
type Time
	func Date(year int, month Month, day, hour, min, sec, nsec int, loc *Location) Time
	func Now() Time
	func Parse(layout, value string) (Time, error)
	func ParseInLocation(layout, value string, loc *Location) (Time, error)
	func Unix(sec int64, nsec int64) Time
	func (t Time) Add(d Duration) Time
	func (t Time) AddDate(years int, months int, days int) Time
	func (t Time) After(u Time) bool
	func (t Time) AppendFormat(b []byte, layout string) []byte
	func (t Time) Before(u Time) bool
	func (t Time) Clock() (hour, min, sec int)
	func (t Time) Date() (year int, month Month, day int)
	func (t Time) Day() int
	func (t Time) Equal(u Time) bool
	func (t Time) Format(layout string) string
	func (t *Time) GobDecode(data []byte) error
	func (t Time) GobEncode() ([]byte, error)
	func (t Time) Hour() int
	func (t Time) ISOWeek() (year, week int)
	func (t Time) In(loc *Location) Time
	func (t Time) IsZero() bool
	func (t Time) Local() Time
	func (t Time) Location() *Location
	func (t Time) MarshalBinary() ([]byte, error)
	func (t Time) MarshalJSON() ([]byte, error)
	func (t Time) MarshalText() ([]byte, error)
	func (t Time) Minute() int
	func (t Time) Month() Month
	func (t Time) Nanosecond() int
	func (t Time) Round(d Duration) Time
	func (t Time) Second() int
	func (t Time) String() string
	func (t Time) Sub(u Time) Duration
	func (t Time) Truncate(d Duration) Time
	func (t Time) UTC() Time
	func (t Time) Unix() int64
	func (t Time) UnixNano() int64
	func (t *Time) UnmarshalBinary(data []byte) error
	func (t *Time) UnmarshalJSON(data []byte) error
	func (t *Time) UnmarshalText(data []byte) error
	func (t Time) Weekday() Weekday
	func (t Time) Year() int
	func (t Time) YearDay() int
	func (t Time) Zone() (name string, offset int)
type Timer
	func AfterFunc(d Duration, f func()) *Timer
	func NewTimer(d Duration) *Timer
	func (t *Timer) Reset(d Duration) bool
	func (t *Timer) Stop() bool
type Weekday
	func (d Weekday) String() string
*/

func main() {
    /*
    	Since返回从t到现在经过的时间，等价于time.Now().Sub(t)
    *func Since(t Time) Duration
        Weekday返回由t指定的星期几
    *func (t Time) Weekday() Weekday
    */
}
```

### exec（实现终端命令、脚本调用功能）

```go
/*
Variables
func LookPath(file string) (string, error)
type Cmd
func Command(name string, arg ...string) *Cmd
func CommandContext(ctx context.Context, name string, arg ...string) *Cmd
func (c *Cmd) CombinedOutput() ([]byte, error)
func (c *Cmd) Output() ([]byte, error)
func (c *Cmd) Run() error
func (c *Cmd) Start() error
func (c *Cmd) StderrPipe() (io.ReadCloser, error)
func (c *Cmd) StdinPipe() (io.WriteCloser, error)
func (c *Cmd) StdoutPipe() (io.ReadCloser, error)
func (c *Cmd) String() string
func (c *Cmd) Wait() error
type Error
func (e *Error) Error() string
func (e *Error) Unwrap() error
type ExitError
func (e *ExitError) Error() string
*/
```

```go
func main() {
    /*
    	Command返回Cmd结构以执行具有给定参数的命名程序
    *func Command(name string,arg ...string) *cmd
    */
    
    /*
    	CommandContext类似于Command，但包含上下文，若上下文在命令本身完成之前完成，则提供的上下文用于终止进程（通过调用os.Process.Kill）
    *func CommandContext(ctx context.Context,name string,arg ...string) *Cmd
    */
    ctx, cancel := context.WithTimeout(context.Background(), 100*time.Millisecond)
	defer cancel()

	if err := exec.CommandContext(ctx, "sleep", "5").Run(); err != nil {
		// This will fail after 100 milliseconds. The 5 second sleep
		// will be interrupted.
	}
    
    /*
    	CombinedOutput运行命令并返回其组合的标准输出和标准错误
    *func (c *Cmd) CombineOutput() ([]byte,error)
    */
    
}
```