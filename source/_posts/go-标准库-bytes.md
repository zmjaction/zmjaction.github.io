---
title: go -- 标准库-- bytes.buffer
mathjax: false
date: 2018-03-02 13:08:18
categories:
    - go
    - bytes
    - buffer
tags:
    - go
    - bytes
    - buffer
---

bytes.buffer是一个缓冲byte类型的缓冲器

1、使用bytes.NewBuffer创建：参数是[]byte的话，缓冲器里就是这个slice的内容；如果参数是nil的话，就是创建一个空的缓冲器。

2、bytes.NewBufferString创建

```go
func main(){
   buf1 := bytes.NewBufferString("hello")
   buf2 := bytes.NewBuffer([]byte("hello"))
   buf3 := bytes.NewBuffer([]byte{'h','e','l','l','o'})
   以上三者等效,输出//hello
   buf4 := bytes.NewBufferString("")
   buf5 := bytes.NewBuffer([]byte{})
   以上两者等效,输出//""
   fmt.Println(buf1.String(),buf2.String(),buf3.String(),buf4,buf5,1)
}
```

<!-- more -->

### 二、写入到缓冲器

如果buffer在new的时候是空的，可以用Write在尾部写入

1、Write方法，将一个byte类型的slice放到缓冲器的尾部

```go
//func (b *Buffer) Write(p []byte) (n int,err error)

func main(){
   s := []byte(" world")
   buf := bytes.NewBufferString("hello") 
   fmt.Println(buf.String())    //hello
   buf.Write(s)                 //将s这个slice添加到buf的尾部
   fmt.Println(buf.String())   //hello world
}

```

2、WriteString方法，把一个字符串放到缓冲器的尾部

```go
//func (b *Buffer) WriteString(s string)(n int,err error)

func main(){
   s := " world"
   buf := bytes.NewBufferString("hello")
   fmt.Println(buf.String())    //hello
   buf.WriteString(s)           //将string写入到buf的尾部
   fmt.Println(buf.String())    //hello world
}

```

3、WriteByte方法，将一个byte类型的数据放到缓冲器的尾部

```go
//func (b *Buffer) WriteByte(c byte) error

func main(){
   var s byte = '?'
   buf := bytes.NewBufferString("hello") 
   fmt.Println(buf.String()) //把buf的内容转换为string，hello
   buf.WriteByte(s)         //将s写到buf的尾部
   fmt.Println(buf.String()) //hello？
}
```

4、WriteRune方法，将一个rune类型的数据放到缓冲器的尾部

```go
// func (b *Buffer) WriteRune(r Rune) (n int,err error)

func main(){
   var s rune = '好'
   buf := bytes.NewBufferString("hello")
   fmt.Println(buf.String()) //hello
   buf.WriteRune(s)   
   fmt.Println(buf.String()) //hello好
}
```

### 三、从缓冲器写出

WriteTo方法，将一个缓冲器的数据写到w里，w是实现io.Writer的，比如os.File

```go
func main(){
   file,_ := os.Create("text.txt")
   buf := bytes.NewBufferString("hello world")
   buf.WriteTo(file)
   //或者使用写入，fmt.Fprintf(file,buf.String())
}
```

### 四、读出缓冲器

1、Read方法，给Read方法一个容器，读完后p就满了，缓冲器相应的减少。

```go
// func (b *Buffer) Read(p []byte)(n int,err error)

func main(){
   s1 := []byte("hello")
   buff := bytes.NewBuffer(s1)
   s2 := []byte(" world")
   buff.Write(s2)
   fmt.Println(buff.String()) //hello world
   
   s3 := make([]byte,3)
   buff.Read(s3)     //把buff的内容读入到s3，s3的容量为3，读了3个过来
   fmt.Println(buff.String()) //lo world
   fmt.Println(string(s3))   //hel
   buff.Read(s3) //继续读入3个，原来的被覆盖
   
   fmt.Println(buff.String())     //world
   fmt.Println(string(s3))    //"lo "
}

```

2、ReadByte方法，返回缓冲器头部的第一个byte，缓冲器头部第一个byte取出

```go
//func (b *Buffer) ReadByte() (c byte,err error)

func main(){
   buf := bytes.NewBufferString("hello")
   fmt.Println(buf.String())
   b,_ := buf.ReadByte()   //取出第一个byte，赋值给b
   fmt.Println(buf.String()) //ello
   fmt.Println(string(b))   //h
}

```

3、ReadRune方法，返回缓冲器头部的第一个rune

```go
// func (b *Buffer) ReadRune() (r rune,size int,err error)

func main(){
   buf := bytes.NewBufferString("你好smith")
   fmt.Println(buf.String())
   b,n,_ := buf.ReadRune()  //取出第一个rune
   fmt.Println(buf.String()) //好smith
   fmt.Println(string(b))   //你
   fmt.Println(n)   //3,"你“作为utf8存储占3个byte

   b,n,_ = buf.ReadRune()  //再取出一个rune
   fmt.Println(buf.String()) //smith
   fmt.Println(string(b))  //好
   fmt.Println(n)   //3
}

```

4、ReadBytes方法，需要一个byte作为分隔符，读的时候从缓冲器里找出第一个出现的分隔符，缓冲器头部开始到分隔符之间的byte返回。

```go
//func (b *Buffer) ReadBytes(delim byte) (line []byte,err error)

func main(){
   var d byte = 'e'  //分隔符
   buf := bytes.NewBufferString("你好esmieth")
   fmt.Println(buf.String()) //你好esmieth
   b,_ := buf.ReadBytes(d)  //读到分隔符，并返回给b
   fmt.Println(buf.String())  //smieth
   fmt.Println(string(b)) //你好e
}

```

5、ReadString方法，和ReadBytes方法一样

```go
//func (b *Buffer) ReadString(delim byte) (line string,err error)

func main(){
   var d byte = 'e'
   buf := bytes.NewBufferString("你好esmieth")
   fmt.Println(buf.String())  //你好esmieth
   b,_ := buf.ReadString(d)   //读取到分隔符，并返回给b
   fmt.Println(buf.String()) //smieth
   fmt.Println(string(b)) //你好e
}

```

### 五、读入缓冲器

ReadFrom方法，从一个实现io.Reader接口的r，把r的内容读到缓冲器里，n返回读的数量

```go
//func (b *Buffer) ReadFrom(r io.Reader) (n int64,err error)

func main(){
   file, _ := os.Open("text.txt")
   buf := bytes.NewBufferString("bob ")
   buf.ReadFrom(file)
   fmt.Println(buf.String()) //bob hello world
}
```

### 六、从缓冲器取出

Next方法，返回前n个byte（slice），原缓冲器变小

```go
//func (b *Buffer) Next(n int) []byte

func main(){
   buf := bytes.NewBufferString("hello world")
   fmt.Println(buf.String())
   b := buf.Next(2)  //取前2个
   fmt.Println(buf.String()) //llo world
   fmt.Println(string(b)) //he
}

```

