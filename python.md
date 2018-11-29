### 查询zabbix数据库

1、获取zabbix的组，主机，键值，value\_type 的sql

```
select   
        c.name as 主机组名, 
        a.name as 主机名,  
        a.hostid as 主机编号,   
        d.name as 键值名,  
        d.itemid as 键值编号, 
        d.key_ as 键值,   
        d.value_type    
        FROM
                hosts a
        INNER JOIN hosts_groups b  ON (a.hostid = b.hostid)
        INNER JOIN groups  c ON (b.groupid = c.groupid )
        RIGHT JOIN items d on (a.hostid = d.hostid)

        where trim(d.error) ='' and a.status != 3 and a.status !=1 and 
        c.name like %s  and d.key_ not like %s and d.key_ not like %s
```

其中第一个%s限定查询的组的名称

第二个%s和第三个 排除自动发现的键值，因为这些键值没有数值的。

hosts表中的status代表监控主机的状态，1代表监控没有启用，3代表模板，所以需要排除掉

2、相关python脚本

```
#! /usr/bin/env python
# -*- coding: utf-8 -*-
# Author:brady



#用来读取 主机组名    主机名    主机编号    键值名    键值编号    键值 健对应的最新值


import os
import  pymysql
from openpyxl import Workbook
from openpyxl import load_workbook

def sqlconn(sqlSten,group,disCov1='',disCov2=''):
  #设置对应的zabbix数据库的相关信息
    db = pymysql.connect(host="10.10.10.1",user="zabbix",passwd="zabbix",db="zabbix",charset="utf8")
    cursor = db.cursor()
    # group = "江苏中烟-%"
    # status = 3
    sql = sqlSten
    print(sql)
    try:
        #执行sql语句
        cursor.execute(sql,(group,disCov1,disCov2))
        # print(cursor.rownumber)
        result = cursor.fetchall()
        # print(result)

    except Exception as e:
        print(e)
    cursor.close()
    db.close()
    return result
def sqlconn1(sqlSten,itemid):
    db = pymysql.connect(host="10.43.242.82",user="zabbix",passwd="zabbix",db="zabbix",charset="utf8")
    cursor = db.cursor()
    # group = "江苏中烟-%"
    # status = 3
    sql = sqlSten
    print(sql)
    try:
        #执行sql语句
        cursor.execute(sql,(itemid))
        # print(cursor.rownumber)
        result = cursor.fetchall()
        # print(result)

    except Exception as e:
        print(e)
    cursor.close()
    db.close()
    return result

def write_excel(data):
    #获取excel路径
    excelPath = os.getcwd()
    print("=============")
    print(excelPath)
    #定义文件名称

    excelName = 'jszydata' + '.xlsx'
    excelFullName = os.path.join(excelPath,excelName)
    wb = Workbook()
    ws = wb.active
    #设置表头
    tableTitle = ['主机组名','主机名','主机编号','键值名','键值编号','键值','value_type','value']
    for col in range(len(tableTitle)):
        c = col + 1
        ws.cell(row=1,column=c).value = tableTitle[col]
    tableValues = data
    for row in range(len(tableValues)):
        ws.append(tableValues[row])
    wb.save(filename=excelFullName)
    return excelFullName

def read_excel(excelFullName):
    wb = load_workbook(excelFullName)
    sheets = wb.get_sheet_names()
    print(sheets)
    sheetFirst = sheets[0]
    ws = wb.get_sheet_by_name(sheetFirst)
    print("====================")
    print(sheetFirst)
    print(ws.title)
    rows = ws.rows
    print(rows)

def main():
    #查询 主机组名','主机名','主机编号','键值名','键值编号','键值','value_type'的sql
    message = '''select   
        c.name as 主机组名, 
        a.name as 主机名,  
        a.hostid as 主机编号,   
        d.name as 键值名,  
        d.itemid as 键值编号, 
        d.key_ as 键值,   
        d.value_type    
        FROM
                hosts a
        INNER JOIN hosts_groups b  ON (a.hostid = b.hostid)
        INNER JOIN groups  c ON (b.groupid = c.groupid )
        RIGHT JOIN items d on (a.hostid = d.hostid)

        where trim(d.error) ='' and a.status != 3 and a.status !=1 and c.name like %s  and d.key_ not like %s and d.key_ not like %s'''
    jszyData = sqlconn(sqlSten=message,group="group-%",disCov1="%.discovery",disCov2="%[{#%")
    dataList = []
    print(jszyData)
    for  i in jszyData:
        #将tuple改为list
        tupleList = list(i)
        print(tupleList)
        valueType = tupleList[-1]

        if valueType == 0:
            sql0 = "select value from history where itemid = %s order by clock desc limit 1"
            nowValue = sqlconn1(sqlSten=sql0,itemid=tupleList[4])
            #判断是否查询到的实时值，如果没有，就赋值为not value
            if len(nowValue):
                value = nowValue[0][0]
                print(value), type(value)
            else:
                value = "not value"
        elif valueType == 1:
            sql1 = "select value from history_str where itemid = %s order by clock desc limit 1"
            nowValue = sqlconn1(sqlSten=sql1,itemid=tupleList[4])
            if len(nowValue):
                value = nowValue[0][0]
                print(value), type(value)
            else:
                value = "not value"
        elif valueType == 2:
            sql2 = "select value from history_log where itemid = %s order by clock desc limit 1"
            nowValue = sqlconn1(sqlSten=sql2,itemid=tupleList[4])
            if len(nowValue):
                value = nowValue[0][0]
                print(value), type(value)
            else:
                value = "not value"
        elif valueType == 3:
            sql3 = "select value from history_uint where itemid = %s order by clock desc limit 1"
            nowValue = sqlconn1(sqlSten=sql3,itemid=tupleList[4])
            if len(nowValue):
                value = nowValue[0][0]
                print(value), type(value)
            else:
                value = "not value"

        elif valueType == 4:
            sql4 = "select value from history_text where itemid = %s order by clock desc limit 1"
            nowValue = sqlconn1(sqlSten=sql4,itemid=tupleList[4])
            if len(nowValue):
                value = nowValue[0][0]
                print(value), type(value)
            else:
                value = "not value"

        else:
            print("数据出现其他错误")
        tupleList.append(value)
        print(tupleList)
        dataList.append(tupleList)
    print(dataList)
    return dataList
if __name__ == '__main__':
    dataZab = main()
    write_excel(data=dataZab)
```









