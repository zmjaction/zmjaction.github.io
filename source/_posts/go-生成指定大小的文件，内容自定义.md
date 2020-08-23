---
title: go -- 生成指定大小文件
mathjax: false
date: 2020-08-21 10:31:18
categories:
    - go
    - go-script
tags:
    - go
    - go-script
---
```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"math"
	"os"
	"strings"
)

func main() {
	s := map[string]interface{}{
		"name": "harry",
		"age": 20,
	}
	sStr, err := json.Marshal(s)
	fmt.Println(len(sStr))
	if err != nil {
		fmt.Println(err)
	}
	CreateFixedFile(4*1024, string(sStr), "./small.txt")
}

// 创建指定大小文件
func CreateFixedFile(size float64, source string, fileName string) {
	file, err := os.OpenFile(fileName, os.O_CREATE|os.O_WRONLY|os.O_APPEND, os.ModePerm)
	if err != nil {
		fmt.Println(err)
	}
	defer file.Close()

	count := math.Ceil(float64(size) / float64(len(source)))

	fmt.Println(count)
	n, err := file.WriteString(strings.Repeat(source+"," + "\r\n", int(count)) )
	HandleErr(err)
	fmt.Println(n)

}

// 错误处理
func HandleErr(err error) {
	if err != nil {
		log.Fatal(err)
	}
}
```