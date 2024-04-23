---
title: golang封装csv数据库
link: golang封装csv数据库
categories:
- Golang
date: 2024-04-17 18:06:58 +0800
date_modified: 2024-04-17 18:06:58 +0800
---

## 基础封装

```
/*************************************************************************
> File Name:     csvData.go
> Author:        XuHao
> Created Time:  2024/4/17 16:38
 ************************************************************************/

package utils

import (
	"encoding/csv"
	"errors"
	"fmt"
	"log"
	"os"
	"reflect"
)

type Book struct {
	ID     string `shasha:"id"`
	Title  string `shasha:"title"`
	Author string `shasha:"author"`
	Year   string `shasha:"year"`
}

type Database struct {
	RootDir string
}

func NewDatabase(rootDir string) *Database {
	return &Database{
		RootDir: rootDir,
	}
}

func (db *Database) CreateTable(databaseName, tableName string, data interface{}) error {
	tablePath := db.getFilePath(databaseName, tableName)
	err := MkDir(db.RootDir + "/" + databaseName)
	file, err := os.OpenFile(tablePath, os.O_CREATE|os.O_RDWR, 0644)
	if err != nil {
		return err
	}
	defer file.Close()

	writer := csv.NewWriter(file)
	defer writer.Flush()

	val := reflect.ValueOf(data)
	typ := reflect.TypeOf(data)

	var columns []string
	for i := 0; i < val.NumField(); i++ {
		tag := typ.Field(i).Tag.Get("shasha")
		columns = append(columns, tag)
	}

	err = writer.Write(columns)
	if err != nil {
		return err
	}

	log.Printf("Table %s created successfully in database %s\n", tableName, databaseName)
	return nil
}

func (db *Database) getFilePath(databaseName, tableName string) string {
	return fmt.Sprintf("%s/%s/%s.csv", db.RootDir, databaseName, tableName)
}

func (db *Database) GetRow(databaseName, tableName, key, value string) ([]string, error) {
	file, err := os.Open(db.getFilePath(databaseName, tableName))
	if err != nil {
		return nil, err
	}
	defer file.Close()

	reader := csv.NewReader(file)

	records, err := reader.ReadAll()
	if err != nil {
		return nil, err
	}

	for _, record := range records {
		if record[2] == value {
			return record, nil
		}
	}

	return nil, fmt.Errorf("Row not found with key %s and value %s", key, value)
}

func (db *Database) GetRows(databaseName, tableName string) ([][]string, error) {
	file, err := os.Open(db.getFilePath(databaseName, tableName))
	if err != nil {
		return nil, err
	}
	defer file.Close()

	reader := csv.NewReader(file)

	records, err := reader.ReadAll()
	if err != nil {
		return nil, err
	}

	if len(records) == 0 {
		return nil, errors.New("Rows not found")
	}

	return records, nil
}

func (db *Database) DeleteRow(databaseName, tableName, key, value string) error {
	file, err := os.Open(db.getFilePath(databaseName, tableName))
	if err != nil {
		return err
	}
	defer file.Close()

	records, err := db.getRecords(file)
	if err != nil {
		return err
	}

	var updatedRecords [][]string
	for _, record := range records {
		if record[2] != value {
			updatedRecords = append(updatedRecords, record)
		}
	}

	err = db.writeRecords(databaseName, tableName, updatedRecords)
	if err != nil {
		return err
	}

	return nil
}

func (db *Database) UpdateRow(databaseName, tableName, key, value string, newData []string) error {
	file, err := os.Open(db.getFilePath(databaseName, tableName))
	if err != nil {
		return err
	}
	defer file.Close()

	records, err := db.getRecords(file)
	if err != nil {
		return err
	}

	var updatedRecords [][]string
	for _, record := range records {
		if record[2] == value {
			updatedRecords = append(updatedRecords, newData)
		} else {
			updatedRecords = append(updatedRecords, record)
		}
	}

	err = db.writeRecords(databaseName, tableName, updatedRecords)
	if err != nil {
		return err
	}

	return nil
}

//
// writeRecords
//  @Description: 用于更新或删除
//  @receiver db
//  @param databaseName
//  @param tableName
//  @param records
//  @return error
//

func (db *Database) writeRecords(databaseName, tableName string, records [][]string) error {
	file, err := os.OpenFile(db.getFilePath(databaseName, tableName), os.O_CREATE|os.O_RDWR|os.O_TRUNC, 0644)
	if err != nil {
		return err
	}
	defer file.Close()

	writer := csv.NewWriter(file)
	defer writer.Flush()

	for _, record := range records {
		err := writer.Write(record)
		if err != nil {
			return err
		}
	}

	return nil
}

// Insert
//
//	@Description: 只插入单条
//	@receiver db
//	@param databaseName
//	@param tableName
//	@param data
//	@return error
func (db *Database) Insert(databaseName, tableName string, obj interface{}) error {
	tablePath := db.getFilePath(databaseName, tableName)

	file, err := os.OpenFile(tablePath, os.O_WRONLY|os.O_APPEND, 0644)
	if err != nil {
		return err
	}
	defer file.Close()

	writer := csv.NewWriter(file)
	defer writer.Flush()

	record := make([]string, 0)

	t := reflect.TypeOf(obj)
	v := reflect.ValueOf(obj)

	for i := 0; i < t.NumField(); i++ {
		record = append(record, v.Field(i).String())
	}

	err = writer.Write(record)
	if err != nil {
		return err
	}

	fmt.Printf("Data inserted successfully into table %s in database %s\n", tableName, databaseName)

	return nil
}

func (db *Database) getRecords(file *os.File) ([][]string, error) {
	reader := csv.NewReader(file)

	records, err := reader.ReadAll()
	if err != nil {
		return nil, err
	}

	return records, nil
}

```

## 测试用例
```
/*************************************************************************
> File Name:     csvData_test.go
> Author:        XuHao
> Created Time:  2024/4/17 16:44
 ************************************************************************/

package utils

import (
	"fmt"
	"testing"
)

func Test_main(t *testing.T) {
	db := NewDatabase(".")
	//
	//book := Book{
	//	ID:     "1",
	//	Title:  "Golang Programming",
	//	Author: "John Doe1",
	//	Year:   "2021",
	//}

	//err := db.CreateTable("goLib", "books", book)
	//if err != nil {
	//	fmt.Println("Error creating table:", err)
	//}

	//err = db.Insert("goLib", "books", book)
	// 查询单条记录
	row, err := db.GetRow("goLib", "books", "id", "John Doe")
	if err != nil {
		fmt.Println("Error querying row:", err)
	} else {
		fmt.Println("Single Row:", row)
	}
	//
	//// 查询多条记录
	//rows, err := db.GetRows("library", "books", "author", "John Doe")
	//if err != nil {
	//	fmt.Println("Error querying rows:", err)
	//} else {
	//	fmt.Println("Multiple Rows:", rows)
	//}
	//
	// 更新记录
	newData := []string{"1", "Python Programming", "Jane Doe4445", "2023"}
	err = db.UpdateRow("goLib", "books", "id", "Jane Doe2", newData)
	if err != nil {
		fmt.Println("Error updating row:", err)
	}
	//
	//// 删除记录
	//err = db.DeleteRow("goLib", "books", "id", "John Doe1")
	//if err != nil {
	//	fmt.Println("Error deleting row:", err)
	//}
}

```