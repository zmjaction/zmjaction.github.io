---
title: go -- excelize-- 导出文件
mathjax: false
date: 2020-07-06 10:10:09
categories:
    - go
    - excelize
tags:
    - go
    - excelize
---

## 360EntSecGroup-Skylar/excelize

### 安装

```
go get github.com/360EntSecGroup-Skylar/excelize
```

#### Gin 中导出 Excel

```go
type ExportField struct {
	phone_num sql.NullInt64 `json:"phone_num"`
	intent_level sql.NullInt64
	call_result sql.NullInt64
	call_start_time sql.NullString
	call_end_time sql.NullString
	talk_duration sql.NullInt64
	r_time  sql.NullString
	calling_number  sql.NullString
	hang_up_note  sql.NullString
}
func main() {
	engine := gin.Default()
	engine.GET("/export", func(c *gin.Context) {
		head := []string{"序号","预警意向", "号码", "完成状态", "开始时间", "结束时间", "通话时长", "通话状态", "主叫号码", "上传时间", "挂机短信"}
		query_sql := `
			SELECT
				phone_num,
				intent_level,
				call_result,
				to_char(
					call_start_time,
					'YYYY-MM-DD hh24:mi:ss'
				) AS call_start_time,
				to_char(
					call_end_time,
					'YYYY-MM-DD hh24:mi:ss'
				) AS call_end_time,
				talk_duration,
				to_char(
					r_time,
					'YYYY-MM-DD hh24:mi:ss'
				) AS r_time,
			calling_number, hang_up_note
			FROM
				aicall.sbc_task_list`
		rows, err := postgresql.DBConnPostGresql().Query(query_sql)
		if err != nil {
			fmt.Println(err)

		}
		var alldata [][]interface{}
		var num int
		for rows.Next(){
			var export ExportField
			err := rows.Scan(&export.phone_num, &export.intent_level, &export.call_result, &export.call_start_time,
				&export.call_end_time, &export.talk_duration, &export.r_time, &export.calling_number, &export.hang_up_note)
			if err != nil {
				fmt.Println(err)
			}
			data := []interface{}{
				num,
				export.intent_level.Int64,
				export.phone_num.Int64,
				export.call_result.Int64,
				export.call_start_time.String,
				export.call_end_time.String,
				export.talk_duration.Int64,
				export.call_result.Int64,
				export.calling_number.String,
				export.r_time.String,
				export.hang_up_note.String,
			}
			alldata = append(alldata, data)
		}
		filename := "task" + "_" + time.Now().Format("20060102150405") + ".xlsx"
		Export(c, head, alldata, filename)

	})
	engine.Run(":9900")
}

func Export(c *gin.Context, head []string, body [][]interface{}, filename string) {
	xlsx := excelize.NewFile()
	xlsx.SetSheetRow("Sheet1", "A1", &head)
	for index, rowData :=  range body{
		xlsx.SetSheetRow("Sheet1", "A" + strconv.Itoa(index + 2), &rowData) // SetSheetRow：设置一行数据 SetCellValue：设置一个数据
	}
	c.Header("Content-Type", "application/octet-stream")
	c.Header("Content-Disposition", "attachment; filename=" + filename)
	c.Header("Content-Transfer-Encoding", "binary")
	_ = xlsx.Write(c.Writer)

}
```

