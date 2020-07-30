---
title: Go--发起GET请求和POST请求
date: 2018-03-12 03:21:41
categories:
    - Go
    - Get
    - Post
tags:
    - Go
    - Get
    - Post
---
### Golang 发起GET请求
#### 基本的GET请求
```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"net/http/httputil"
)

func main() {
	resp, err := http.Get("http://httpbin.org/get")
	if err != nil {
		log.Fatal(err)
		return
	}
	defer resp.Body.Close()
	response, err := httputil.DumpResponse(resp, true)
	if err != nil {
		panic(err)
	}
	fmt.Println(string(response))

}

```
#### 带有参数的GET请求
```go
package main

import (
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
)

func main() {
	resp, err := http.Get("http://httpbin.org/get?name=tom&age=19")
	if err != nil {
		log.Fatal(err)
		return
	}
	defer resp.Body.Close()
	respone, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
		return
	}
	fmt.Println(string(respone))

}

```

#### 动态传递GET请求参数
```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
	"net/url"
)

func main(){
	params := url.Values{}
	Url, err := url.Parse("http://httpbin.org/get")
	if err != nil {
		return
	}
	params.Set("name","xiaoming")
	params.Set("age","19")
	Url.RawQuery = params.Encode()  //如果参数中有中文参数,这个方法会进行URLEncode
	urlPath := Url.String()  // http://httpbin.org/get?age=19&name=xiaoming{
	resp,err := http.Get(urlPath)
	defer resp.Body.Close()
	body, _ := ioutil.ReadAll(resp.Body)
	fmt.Println(string(body))
}
```

#### GET请求json返回值
```go

package main

import (
	"encoding/json"
	"fmt"
	"io/ioutil"
	"net/http"
)

type result struct {
	Args string `json:"args"`
	Headers map[string]string `json:"headers"`
	Origin string `json:"origin"`
	Url string `json:"url"`
}

func main(){
	resp, err := http.Get("http://httpbin.org/get")
	if err != nil {
		panic(err)
	}
	defer resp.Body.Close()
	body, err := ioutil.ReadAll(resp.Body)
	fmt.Println(string(body))
	var res result
	_ = json.Unmarshal(body, &res)
	fmt.Printf("%#v", res)

}
```
#### GET请求添加请求头

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
)

func main(){
	client := &http.Client{}
	request, err := http.NewRequest("GET", " http://httpbin.org/get", nil)
	if err != nil {
		fmt.Println(err)
	}
	request.Header.Add("User-Agent", "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.116 Safari/537.36")
	request.Header.Add("name", "xiaoming")
	resp, _ := client.Do(request)
	body, _ := ioutil.ReadAll(resp.Body)

	fmt.Println(string(body))
}
```
### Golang 发起POST请求
#### (1)基本 POST 请求
```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
	"net/url"
)

func main(){
	values := url.Values{}
	values.Add("username", "xiaoming")
	values.Add("pwd", "pwd")
	resp, _ := http.PostForm("http://httpbin.org/post", values)
	body, _ := ioutil.ReadAll(resp.Body)
	fmt.Println(string(body))
}
```
#### (2)基本的post请求
```go

package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
	"net/url"
	"strings"
)

func main(){
	values := url.Values{
		"word":{"hello"},
		"gender":{"male"},
	}
	encode := values.Encode()
	resp, err := http.Post("http://httpbin.org/post", "text/html", strings.NewReader(encode))
	if err != nil {
		panic(err)
	}
	body, err := ioutil.ReadAll(resp.Body)
	fmt.Println(string(body))
}
```

#### (1)POST请求 发送json数据
```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"net/http"
)

func main(){
	client := &http.Client{}
	data := make(map[string]interface{})
	data["username"] = "xiaoming"
	data["pwd"] = "pwd"
	reqData, _ := json.Marshal(data)
	req, _ := http.NewRequest("POST", "http://httpbin.org/post", bytes.NewReader(reqData))
	resp, _ := client.Do(req)
	all, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		panic(err)
	}
	fmt.Println(string(all))
}

```
#### (2)POST请求 发送json数据
```go 

package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"net/http"
)

func main(){
	data := make(map[string]interface{})
	data["username"] = "xiaoming"
	data["pwd"] = "pwd"
	reqData, _ := json.Marshal(data)
	req, _ := http.Post("http://httpbin.org/post","application/json", bytes.NewReader(reqData))
	all, err := ioutil.ReadAll(req.Body)
	if err != nil {
		panic(err)
	}
	fmt.Println(string(all))
}
```


















