
---
title: 用GOB传输数据
mathjax: false
date: 2017-04-11 08:15:51
categories:
    - go
tags:
    - go
    - GOB
---

示例代码：
##### go 编码到文件，读取文件内容并解码
```go
package main

import (
	"bufio"
	"encoding/gob"
	"fmt"
	"log"
	"os"
)

type Address struct {
	Type             string
	City             string
	Country          string
}

type VCard struct {
	FirstName	string
	LastName	string
	Addresses	[]*Address
	Remark		string
}


func main() {
	//WriteGobFile()
	ReadGobFile()

}
func WriteGobFile() {
	pa := &Address{"private", "Aartselaar","Belgium"}
	wa := &Address{"work", "Boom", "Belgium"}
	vc := VCard{"Jan", "Kersschot", []*Address{pa,wa}, "none"}
	file, _ := os.OpenFile("vcard.gob", os.O_CREATE|os.O_WRONLY, 0666)
	defer file.Close()
	enc := gob.NewEncoder(file)  // Will write to network.
	err := enc.Encode(vc)
	if err != nil {
		log.Println("Error in encoding gob")
	}
}

func ReadGobFile() {
	file, err := os.Open("vcard.gob")
	if err != nil {
		fmt.Println(err)
	}
	defer file.Close()
	rd := bufio.NewReader(file)
	dec := gob.NewDecoder(rd)
	var v VCard
	err = dec.Decode(&v)
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println(v)

}
```
参考链接：https://github.com/zmjaction/the-way-to-go_ZH_CN/blob/master/eBook/12.11.md
