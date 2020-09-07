---
title: Gin中使用JWT
mathjax: false
date: 2020-08-27 00:15:11
categories:
    - go
    - Gin
tags:
    - go
    - gin
    - JWT
---


## 为什么需要JWT？
在之前的一些web项目中，我们通常使用的是Cookie-Session模式实现用户认证。相关流程大致如下：

1. 用户在浏览器端填写用户名和密码，并发送给服务端
2. 服务端对用户名和密码校验通过后会生成一份保存当前用户相关信息的session数据和一个与之对应的标识（通常称为session_id）
3. 服务端返回响应时将上一步的session_id写入用户浏览器的Cookie
4. 后续用户来自该浏览器的每次请求都会自动携带包含session_id的Cookie
5. 服务端通过请求中的session_id就能找到之前保存的该用户那份session数据，从而获取该用户的相关信息。
 
<!-- more -->

这种方案依赖于客户端（浏览器）保存Cookie，并且需要在服务端存储用户的session数据。

在移动互联网时代，我们的用户可能使用浏览器也可能使用APP来访问我们的服务，我们的web应用可能是前后端分开部署在不同的端口，有时候我们还需要支持第三方登录，这下Cookie-Session的模式就有些力不从心了。

JWT就是一种基于Token的轻量级认证模式，服务端认证通过后，会生成一个JSON对象，经过签名后得到一个Token（令牌）再发回给用户，用户后续请求只需要带上这个Token，服务端解密之后就能获取该用户的相关信息了。

参考：[李文周博客](https://www.liwenzhou.com/posts/Go/jwt_in_gin/) [阮一峰的网络日志](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html) [煎鱼的博客](https://segmentfault.com/a/1190000013297828)

#### 下载依赖包

```bash
go get -u github.com/dgrijalva/jwt-go
```

#### 定义需求

我们需要定制自己的需求来决定JWT中保存哪些数据，比如我们规定在JWT中要存储`username`信息，那么我们就定义一个`MyClaims`结构体如下：

```
// Claims 自定义声明结构体并内嵌jwt.StandardClaims
// jwt包自带的jwt.StandardClaims只包含了官方字段
// 我们这里需要额外记录一个username字段，所以要自定义结构体
// 如果想要保存更多信息，都可以添加到这个结构体中

type Claims struct {
    Username string `json:"username"`
    Password string `json:"password"`
    jwt.StandardClaims
}
```

然后我们定义JWT的过期时间，这里以2小时为例：

```
const TokenExpireDuration = time.Hour * 2
```

接下来还需要定义Secret：

```
var jwtSecret = []byte(setting.JwtSecret)
```

#### 生成JWT

```go
func GenerateToken(username, password string) (string, error) {
    claims := Claims{
        username,
        password,
        jwt.StandardClaims {
            ExpiresAt : time.Now().Add(TokenExpireDuration).Unix(),
            Issuer : "gin-blog",
        },
    }
	// 使用指定的签名方法创建签名对象
    tokenClaims := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    // 使用指定的secret签名并获得完整的编码后的字符串token
    token, err := tokenClaims.SignedString(jwtSecret)

    return token, err
}
```

#### 解析JWT

```go
func ParseToken(token string) (*Claims, error) {
    // 解析token
    tokenClaims, err := jwt.ParseWithClaims(token, &Claims{}, func(token *jwt.Token) (interface{}, error) {
        return jwtSecret, nil
    })

    if tokenClaims != nil {
        if claims, ok := tokenClaims.Claims.(*Claims); ok && tokenClaims.Valid {
            return claims, nil
        }
    }

    return nil, err
}
```

- `NewWithClaims(method SigningMethod, claims Claims)`，`method`对应着`SigningMethodHMAC struct{}`，其包含`SigningMethodHS256`、`SigningMethodHS384`、`SigningMethodHS512`三种`crypto.Hash`方案
- `func (t *Token) SignedString(key interface{})` 该方法内部生成签名字符串，再用于获取完整、已签名的`token`
- `func (p *Parser) ParseWithClaims` 用于解析鉴权的声明，[方法内部](https://gowalker.org/github.com/dgrijalva/jwt-go#Parser_ParseWithClaims)主要是具体的解码和校验的过程，最终返回`*Token`
- `func (m MapClaims) Valid()` 验证基于时间的声明`exp, iat, nbf`，注意如果没有任何声明在令牌中，仍然会被认为是有效的。并且对于时区偏差没有计算方法

#### GIn中间件

```go
package jwt

import (
    "time"
    "net/http"

    "github.com/gin-gonic/gin"

    "gin-blog/pkg/util"
    "gin-blog/pkg/e"
)

func JWTAuthMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        var code int
        var data interface{}

        code = e.SUCCESS
        token := c.Query("token")
        if token == "" {
            code = e.INVALID_PARAMS
        } else {
            claims, err := util.ParseToken(token)
            if err != nil {
                code = e.ERROR_AUTH_CHECK_TOKEN_FAIL
            } else if time.Now().Unix() > claims.ExpiresAt {
                code = e.ERROR_AUTH_CHECK_TOKEN_TIMEOUT
            }
        }

        if code != e.SUCCESS {
            c.JSON(http.StatusUnauthorized, gin.H{
                "code" : code,
                "msg" : e.GetMsg(code),
                "data" : data,
            })

            c.Abort()
            return
        }

        c.Next()
    }
}
```

#### 获取TOKEN

```go

func authHandler(c *gin.Context) {
	var user UserInfo
	err := c.ShouldBind(&user)
	if err != nil {
		c.JSON(http.StatusOK, gin.H{
			"code":2001,
			"msg":"无效参数",
		})
		c.Abort()
		return
	}
	if user.Username == "qimi" && user.Password=="123456"{
		tokenString, _ := GenToken(user.Username)
		c.JSON(http.StatusOK, gin.H{
			"code":2000,
			"msg":"success",
			"data":gin.H{"token":tokenString},
		})
		return
	}
	c.JSON(http.StatusOK, gin.H{
		"code":2002,
		"msg":"鉴权失败",
	})
	return
}
```

#### 验证TOKEN

用`GET`方式访问`http:127.0.0.1:8080/login?username=qimi&password=123456` 查看返回值是否正确

```go
{
    "code": 2000,
    "data": {
        "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InFpbWkiLCJleHAiOjE1OTkxMjQ1OTgsImlzcyI6Im15LXByb2plY3QifQ.omnDmr9-wSXjjxIZmIysRu7khVB_ginuY9FUqd40q1w"
    },
    "msg": "success"
}
```

验证方法 路由

```go
func homeHandler(c *gin.Context) {
	//username := c.MustGet("username").(string)
	c.JSON(http.StatusOK, gin.H{
		"code": 2000,
		"msg":  "success",
		//"data": gin.H{"username": username},
	})
}
```

用`GET`方式访问`127.0.0.1:8080/home?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InFpbWkiLCJleHAiOjE1OTkxMjQ1OTgsImlzcyI6Im15LXByb2plY3QifQ.omnDmr9-wSXjjxIZmIysRu7khVB_ginuY9FUqd40q1p` 查看返回值是否正确

```go
{
    "code": 2000,
    "msg": "success"
}
```

测试demo

```go
package main

import (
	"errors"
	"fmt"
	"github.com/dgrijalva/jwt-go"
	"github.com/gin-gonic/gin"
	"net/http"
	"time"
)

type UserInfo struct {
	Username string `form:"username" json:"username" binding:"required"`
	Password string `form:"password" json:"password" binding:"required"`
}
type MyClaims struct {
	Username string `json:"username"`
	jwt.StandardClaims
}
const TokenExpireDuration = time.Minute * 2

var MySecret = []byte("夏天夏天悄悄过去")
// GenToken 生成JWT
func GenToken(username string) (string, error) {
	// 创建一个我们自己的声明
	c := MyClaims{
		username, // 自定义字段
		jwt.StandardClaims{
			ExpiresAt: time.Now().Add(TokenExpireDuration).Unix(), // 过期时间
			Issuer:    "my-project",                               // 签发人
		},
	}
	// 使用指定的签名方法创建签名对象
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, c)
	// 使用指定的secret签名并获得完整的编码后的字符串token
	return token.SignedString(MySecret)
}

func authHandler(c *gin.Context) {
	var user UserInfo
	err := c.ShouldBind(&user)
	if err != nil {
		c.JSON(http.StatusOK, gin.H{
			"code":2001,
			"msg":"无效参数",
		})
		c.Abort()
		return
	}
	if user.Username == "qimi" && user.Password=="123456"{
		tokenString, _ := GenToken(user.Username)
		c.JSON(http.StatusOK, gin.H{
			"code":2000,
			"msg":"success",
			"data":gin.H{"token":tokenString},
		})
		return
	}
	c.JSON(http.StatusOK, gin.H{
		"code":2002,
		"msg":"鉴权失败",
	})
	return
}
func ParseToken(tokenString string) (*MyClaims, error) {
	// 解析token
	token, err := jwt.ParseWithClaims(tokenString, &MyClaims{}, func(token *jwt.Token) (i interface{}, err error) {
		return MySecret, nil
	})
	if err != nil {
		return nil, err
	}
	if claims, ok := token.Claims.(*MyClaims); ok && token.Valid { // 校验token
		return claims, nil
	}
	return nil, errors.New("invalid token")
}

//func JWTAuthMiddleware() func(c *gin.Context) {
//	return func(c *gin.Context) {
//		authHeader := c.Request.Header.Get("Authorization")
//		fmt.Println("==========================")
//		fmt.Println(authHeader)
//		fmt.Println("==========================")
//		if authHeader == ""{
//			c.JSON(http.StatusOK, gin.H{
//				"code":2003,
//				"msg":"请求头中auth为空",
//			})
//			c.Abort()
//			return
//		}
//		// 按空格分割
//		parts := strings.SplitN(authHeader, " ", 2)
//		fmt.Println("111111111111111111111", parts)
//		//if !(len(parts) == 2 && parts[0] == "Bearer") {
//		//	c.JSON(http.StatusOK, gin.H{
//		//		"code": 2004,
//		//		"msg":  "请求头中auth格式有误",
//		//	})
//		//	c.Abort()
//		//	return
//		//}
//		// parts[1]是获取到的tokenString，我们使用之前定义好的解析JWT的函数来解析它
//		fmt.Println("===============================")
//		fmt.Println(parts[0])
//		fmt.Println("===============================")
//		mc, err := ParseToken(parts[0])
//		if err != nil {
//			c.JSON(http.StatusOK, gin.H{
//				"code": 2005,
//				"msg":  "无效的Token",
//			})
//			c.Abort()
//			return
//		}
//		// 将当前请求的username信息保存到请求的上下文c上
//		c.Set("username", mc.Username)
//		c.Next() // 后续的处理函数可以用过c.Get("username")来获取当前请求的用户信息
//	}
//
//}
func JWT() gin.HandlerFunc {
	return func(c *gin.Context) {
		var code int
		var data interface{}

		code = 200
		token := c.Query("token")
		if token == "" {
			code = 400
		} else {
			_, err := ParseToken(token)
			if err != nil {
				switch err.(*jwt.ValidationError).Errors {
				case jwt.ValidationErrorExpired:
					code = 500
				default:
					code = 500
				}
			}
		}

		if code != 200 {
			c.JSON(http.StatusUnauthorized, gin.H{
				"code": code,
				"msg":  600,
				"data": data,
			})

			c.Abort()
			return
		}

		c.Next()
	}
}


func homeHandler(c *gin.Context) {
	//username := c.MustGet("username").(string)
	c.JSON(http.StatusOK, gin.H{
		"code": 2000,
		"msg":  "success",
		//"data": gin.H{"username": username},
	})
}
func main() {
	c := gin.Default()
	c.GET("/login", authHandler)
	//c.GET("/home", JWTAuthMiddleware(), homeHandler)
	c.GET("/home", JWT(), homeHandler)

	err := c.Run(":8080")
	if err != nil {
		fmt.Println(err)
	}


}

```

