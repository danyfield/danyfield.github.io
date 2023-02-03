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

### Strings

```go
/*
func Compare(a, b string) int
func Contains(s, substr string) bool
func ContainsAny(s, chars string) bool
func ContainsRune(s string, r rune) bool
func Count(s, substr string) int
func EqualFold(s, t string) bool
func Fields(s string) []string
func FieldsFunc(s string, f func(rune) bool) []string
func HasPrefix(s, prefix string) bool
func HasSuffix(s, suffix string) bool
func Index(s, substr string) int
func IndexAny(s, chars string) int
func IndexByte(s string, c byte) int
func IndexFunc(s string, f func(rune) bool) int
func IndexRune(s string, r rune) int
func Join(elems []string, sep string) string
func LastIndex(s, substr string) int
func LastIndexAny(s, chars string) int
func LastIndexByte(s string, c byte) int
func LastIndexFunc(s string, f func(rune) bool) int
func Map(mapping func(rune) rune, s string) string
func Repeat(s string, count int) string
func Replace(s, old, new string, n int) string
func ReplaceAll(s, old, new string) string
func Split(s, sep string) []string
func SplitAfter(s, sep string) []string
func SplitAfterN(s, sep string, n int) []string
func SplitN(s, sep string, n int) []string
func Title(s string) string
func ToLower(s string) string
func ToLowerSpecial(c unicode.SpecialCase, s string) string
func ToTitle(s string) string
func ToTitleSpecial(c unicode.SpecialCase, s string) string
func ToUpper(s string) string
func ToUpperSpecial(c unicode.SpecialCase, s string) string
func ToValidUTF8(s, replacement string) string
func Trim(s, cutset string) string
func TrimFunc(s string, f func(rune) bool) string
func TrimLeft(s, cutset string) string
func TrimLeftFunc(s string, f func(rune) bool) string
func TrimPrefix(s, prefix string) string
func TrimRight(s, cutset string) string
func TrimRightFunc(s string, f func(rune) bool) string
func TrimSpace(s string) string
func TrimSuffix(s, suffix string) string
type Builder
	func (b *Builder) Cap() int
	func (b *Builder) Grow(n int)
	func (b *Builder) Len() int
	func (b *Builder) Reset()
	func (b *Builder) String() string
	func (b *Builder) Write(p []byte) (int, error)
	func (b *Builder) WriteByte(c byte) error
	func (b *Builder) WriteRune(r rune) (int, error)
	func (b *Builder) WriteString(s string) (int, error)
type Reader
	func NewReader(s string) *Reader
	func (r *Reader) Len() int
	func (r *Reader) Read(b []byte) (n int, err error)
	func (r *Reader) ReadAt(b []byte, off int64) (n int, err error)
	func (r *Reader) ReadByte() (byte, error)
	func (r *Reader) ReadRune() (ch rune, size int, err error)
	func (r *Reader) Reset(s string)
	func (r *Reader) Seek(offset int64, whence int) (int64, error)
	func (r *Reader) Size() int64
	func (r *Reader) UnreadByte() error
	func (r *Reader) UnreadRune() error
	func (r *Reader) WriteTo(w io.Writer) (n int64, err error)
type Replacer
	func NewReplacer(oldnew ...string) *Replacer
	func (r *Replacer) Replace(s string) string
	func (r *Replacer) WriteString(w io.Writer, s string) (n int, err error)
Bugs
*/

```

```go
func main() {
    /*
    	Compare返回一个按字典顺序比较两个字符串的整数，若a == b则为0；若为a < b，结果为-1；若a > b则为1
    	func Compare(a,b string) int
    */
    
    /*
    	判断substr是否在s之内
    	func Contains(s,substr string) bool {
    		return Index(s,substr) >= 0
    	}
    */
    
    /*
    	Count计算s中substr的非重叠实例的数量，若substr是一个空字符串，则Count返回1+s中的Unicode代码点数
    	func Count(s,substr string) int {
    		if len(substr) == 0 {
    			return utf8.RuneCountInString(s) + 1
    		}
    		if len(substr) == 1 {
    			return bytealog.CountString(s,substr[0])
    		}
    		n := 0
    		for {
    			i := Index(s,substr)
    			if i == -1 {return n}
    			n++
    			s = s[i+len(substr):]
    		}
    	}
    */
    
    /*
    	Cap方法返回字节数组分配的内存空间大小
    	func (b *Builder) Cap() int
    	
    	Grow方法扩展buf数组的分配内存的大小
    	func (b *Builder) Grow(n int)
    	
    	Reset方法清空b的所有内容
    	func (b *Builder) Reset() {
    		b.addr = nil
    		b.buf = nil
    	}
    	
    	String方法将b的数据以string类型返回
    	func (b *Builder) String() string {
    		return *(*string)(unsafe.Pointer(&b.buf))
    	}
    */
    b := strings.Builder{}
	_ = b.WriteByte('7')
	fmt.Println(b.String())
}

```

### Strconv

主要提供不同数据类型相互转换的函数和方法

```go
/*
Constants
Variables
func AppendBool(dst []byte, b bool) []byte
func AppendFloat(dst []byte, f float64, fmt byte, prec, bitSize int) []byte
func AppendInt(dst []byte, i int64, base int) []byte
func AppendQuote(dst []byte, s string) []byte
func AppendQuoteRune(dst []byte, r rune) []byte
func AppendQuoteRuneToASCII(dst []byte, r rune) []byte
func AppendQuoteRuneToGraphic(dst []byte, r rune) []byte
func AppendQuoteToASCII(dst []byte, s string) []byte
func AppendQuoteToGraphic(dst []byte, s string) []byte
func AppendUint(dst []byte, i uint64, base int) []byte
func Atoi(s string) (int, error)
func CanBackquote(s string) bool
func FormatBool(b bool) string
func FormatComplex(c complex128, fmt byte, prec, bitSize int) string
func FormatFloat(f float64, fmt byte, prec, bitSize int) string
func FormatInt(i int64, base int) string
func FormatUint(i uint64, base int) string
func IsGraphic(r rune) bool
func IsPrint(r rune) bool
func Itoa(i int) string
func ParseBool(str string) (bool, error)
func ParseComplex(s string, bitSize int) (complex128, error)
func ParseFloat(s string, bitSize int) (float64, error)
func ParseInt(s string, base int, bitSize int) (i int64, err error)
func ParseUint(s string, base int, bitSize int) (uint64, error)
func Quote(s string) string
func QuoteRune(r rune) string
func QuoteRuneToASCII(r rune) string
func QuoteRuneToGraphic(r rune) string
func QuoteToASCII(s string) string
func QuoteToGraphic(s string) string
func Unquote(s string) (string, error)
func UnquoteChar(s string, quote byte) (value rune, multibyte bool, tail string, err error)
type NumError
	func (e *NumError) Error() string
	func (e *NumError) Unwrap() error
*/

```

```go
func main(){
    /*
    	AppendBool根据b的值将"true"或"false"附加到dst返回扩展缓冲区
    *func AppendBool(dst []byte,b bool) []byte {
    	if b {
    		return append(dst, "true"...)
    	}
    	return append(dst, "false"...)
    }
    */
    b := []byte("bool:")
    b = strconv.AppendBool(b,true)
    fmt.Println(string(b))
    
    /*
    	FormatBool根据b的值返回"true"或"false"
    *func FormatBool(b bool) string
    
    	Itoa等效于FormatInt(int64(i),10)
    *func Itoa(i int) string
    */
    
    /*
    	NumError记录转换失败
    *func (e *NumError) Error() string
    */
    str := "Not a number"
    if _,err := strconv.ParseFloat(str,64); err != nil {
        e := err.(*strconv.NumError)
        fmt.Println("Func:", e.Func)
		fmt.Println("Num:", e.Num)
		fmt.Println("Err:", e.Err)
		fmt.Println(err)
    }
}

```

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