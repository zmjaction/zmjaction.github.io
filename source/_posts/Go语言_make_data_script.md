---
title: Go-语言_make_data_script
date: 2020-07-12 23:10:21
categories:
    - Go
    - go_script
tags:
    - Go
    - go_script
---
#### 简单示例
```go
package main

import (
	"fmt"
	"math/rand"
	"net"
	"smartphone/db/postg"

	"time"
)

func main() {
	instr := `insert into aicall.t_rebot_dir (rebot_name,rebot_sign,rebot_state,card_slot,
available_line,normal_line,except_line,device_ip,device_mac) values ($1,$2,$3,$4,$5,$6,$7,$8,$9); `
	rand.Seed(time.Now().UnixNano())
	for i := 0; i < 100; i++ {
		var (
			rebot_state      int
			rebot_name_alias string
		)
		if i%2 == 0 {
			rebot_state = 1
			rebot_name_alias = "外呼中心"
		} else {
			rebot_name_alias = "GOIP"
		}

		card_slot := rand.Intn(30) + 10
		fmt.Println(card_slot)
		except_line := rand.Intn(10)
		fmt.Println(except_line)
		normal_line := card_slot - except_line
		fmt.Println(normal_line)
		rebot_name := fmt.Sprintf("设备%d(%s)", i, rebot_name_alias)
		fmt.Println(rebot_name)
		ip := fmt.Sprintf("%d.%d.%d.%d", rand.Intn(255), rand.Intn(255), rand.Intn(255), rand.Intn(255))
		fmt.Println(ip)
		// 随机mac
		mac := GenerateMac().String()
		// rebot_sign
		rebot_sign := fmt.Sprintf("22948475%d", i)

		fmt.Println(mac)
		ret, err := postg.DBConnPostGresql().Exec(instr,
			rebot_name, rebot_sign, rebot_state, card_slot, card_slot, normal_line, except_line, ip, mac)
		if err != nil {
			fmt.Printf("inster error:%v\n", err)
			return
		}
		rf, _ := ret.RowsAffected()
		fmt.Println(rf)
	}

}
func GenerateMac() net.HardwareAddr {
	buf := make([]byte, 6)
	var mac net.HardwareAddr

	_, err := rand.Read(buf)
	if err != nil {
	}

	// Set the local bit
	//buf[0] = buf[0] | 2 做按位与运算
	buf[0] |= 2

	mac = append(mac, buf[0], buf[1], buf[2], buf[3], buf[4], buf[5])

	return mac
}

```
```go
package main

import (
	"fmt"
	"math/rand"
	"smartphone/db/postg"
	"time"
)

func main() {
	instr := `insert into aicall.t_rebot_dir_detail (rebot_id,line_name,line_state,except_cause) values ($1,$2,$3,$4); `
	query_sql := `
	select id from aicall.t_rebot_dir
`
	rand.Seed(time.Now().UnixNano())
	rows, err := postg.DBConnPostGresql().Query(query_sql)
	if err != nil {
		fmt.Println(err)
	}
	//var allid []int
	for rows.Next() {
		var id int
		rows.Scan(&id)
		//allid = append(allid, id)
		scope := rand.Intn(11) + 2
		for i := 0; i < scope; i++ {
			line_name := fmt.Sprintf("线路%d", i)
			var line_state int
			var except_cause string
			if i%2 == 0 {
				line_state = 1
			} else if i == 7 {
				except_cause = "其他故障"
			} else {
				except_cause = "欠费"
			}
			ret, err := postg.DBConnPostGresql().Exec(instr, id, line_name, line_state, except_cause)
			if err != nil {
				fmt.Println(err)
			}
			rf, _ := ret.RowsAffected()
			fmt.Println(rf)

		}
	}
	//fmt.Println(allid)

}

```