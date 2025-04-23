title: 文件IO
tags: 
	- IO
	- file
date: 2022/06/06
categories: go
---

* 文件一次性全部读取到内存中
	* 通过文件名直接读取文件到内存中
		* os: `func ReadFile(filename string) ([]byte, error)` 
		* os: `func WriteFile(filename string, data []byte, perm FileMode) error`
		* io/ioutil: `func ReadFile(filename string) ([]byte, error)`
		* io/ioutil: `func WriteFIle(filename string, data []byte, perm FileMode) error`
	* 通过文件名获取文件的句柄, 然后通过文件句柄读取文件
		* 获取文件句柄
			* os: `func OpenFile(name string, flag int, perm FileMode) (*File, error)` flag用于指定文件句柄的操作范围，当打开的文件不存在，而flag有os.O_CREATE属性时，就会创建文件，文件的所有者、组、其他人的权限由perm设定
			* os: `func Open(name string) (*File, error)`  Open是OpenFile的简化版，直接读取文件，flag设置为os.O_RDONLY。
		* 通过句柄读取文件
			* os: `func (*File) Read(b []byte) (n int, err error)` 从文件中读取长度为len(b)的字节到b中，返回读取的字节数大小，如果读到文件末尾，则返回0和io.EOF
			* os: `func (*File) ReadAt(b []byte, off int64) (n int, err error)` 从文件开头起偏移off个字节的地方开始，读取长度为len(b)的字节到b中。
* 将文件一部分读入到缓存中,逐份读取
	* io库，定义了这两个接口，并实现了这两个接口的相关函数如`func LimitReader(r Reader, n int64) Reader` n指定要读取的字节数
		* io.Reader, io.Writer接口
		```
		type Reader interface {
			Read(p []byte) (n int, err error)
		}
		type Writer interface {
			Write(p []byte) (n int, err error)
		}
		```
	* bufio在io的基础上实现了带有缓冲的IO,实现了io.Reader,io.Writer接口
	```
	func NewReader(rd io.Reader) *Reader
	func (b *Reader) Read(p []byte) (n int, err error) 
	```
	* 实现io.Reader,io.Writer接口的有：os.File, strings.Reader, net.conn, net.Buffers。os.File 可以通过os.Open()得到。strings.Read 可以通过strings.NewReader()得到。net.conn 可以通过net.Dail()得到
