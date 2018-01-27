---
title: 用python完成自动化任务
date: 2018-01-27 09:01:35
tags:
---

在程序员眼里，见不得不断重复的劳动，恨不得把每天的吃饭睡觉都交给CPU去完成。程序员想偷懒，所以要和计算机进行“亲切友好”的交流，让它能明白要做的事情，这样才能把我们从繁重的劳动中解放出来，要不怎么说懒是推动生产力发展的源动力呢:D

# 前言

今天这事的起源是一个办公场景，有一个Excel表需要核对，表中数据有“课程名”，“教师姓名”，“课程编号”，根据课程编号可以计算出来一个内网下载教学大纲的一个链接，然后检查一下哪些课程没有提交教学大纲，通知相应的老师。

会一点Excel的可能会想到用函数生成链接，然后直接点击链接进行下载，可是下载的文件是以课程编号命名的，还需要写一个根据“课程名”和“教师姓名”进行文件重命名的函数操作，一个50条左右记录的表，手工操作差不多得个把小时吧。看到美好的青春年华被这些繁琐的事务所吞噬，实在忍不住了，看不下去了，看不下去了，看不下去了。于是乎，就有了下面的故事

做个程序吧，这样的任务，应该是很适合用一小段程序来完成。那就试试python吧，至于VBA，后面有时间再弄了，不是说python很开发很快么，那就用这个案例来小试一下。

# 整理下思路

* 在python里操作Excel，引入了`win32com.client`库
* 遍历Excel工作表，获取行记录
* 提取其中的关键信息，生成下载链接
* 下载文件，并按中文信息重命名
* 下载到Excel文件所在的目录中，放入`out\<学期>\`子目录
* 根据下载的情况，在表格中做标记

程序功能简单，没有做界面，通过命令行参数传入Excel文件，这种操作方式对于办公室工作的人员来说，可能不太友好。写个操作说明吧，用几次就会了，比单调的重复劳动应该还是更有效率的。

# 开工

程序需要操作Excel，需要创建目录，需要解析命令行参数，需要下载文件，引入包
```
import urllib3
import os
import sys

from win32com.client import Dispatch
```

*主程序结构*
通过命令行参数获取当前操作的Excel文件，并提取出Excel文件所在的目录
```
if len(sys.argv)<2:
    print("参数不足，退出")
    sys.exit(0)


xlsFile = sys.argv[1]
workFolder = os.path.dirname(sys.argv[1])

print("参数文件：" + xlsFile)
```
Excel文件来源于系统导出，数据结构信息相对固定，直接提取了`sheet1`以及相应的列序号，将表中不太重要的两列数据修改为上传标记和链接地址，下面是打开文件进行遍历的操作
```

xlsApp = Dispatch("Excel.Application")
xlsBook = xlsApp.Workbooks.Open(xlsFile)
xlsSht = xlsBook.Worksheets("sheet1")

xlsSht.Cells(1,22).Value="是否上传"
xlsSht.Cells(1,23).Value="链接地址"


http = urllib3.PoolManager()

for i in range(2,xlsSht.UsedRange.Rows.Count+1):
    courseName = xlsSht.Cells(i,5).Value
    teacher = xlsSht.Cells(i,6).Value
    courseNum = xlsSht.Cells(i,21).Value
    teacher = teacher.replace("/","_")
    print("\n"+str(i)+","+courseName+","+teacher+","+courseNum)

xlsBook.Save()
xlsApp.Application.Quit()

print("运行完毕，OK!")
```
先跑一个，看看能否获得数据。对教师名进行了替换处理，多名老师合上的课程，系统中是用`/`来表示的，而`/`表示路径信息，如果用教师名作为文件名的一个部分，则需要替换为另外的连接符。接下来完成单个的功能，各个零部件。

*生成下载链接*
通过课程编号生成下载链接
```
def genlink(courseNumber):
    '''
    通过课程号生成下载链接地址，课程编号格式如(2017-2018-1)-ECC208-20063065-1
    '''
    year = courseNumber[3:5]+courseNumber[8:10]
    link = "http://192.168.12.39/kcjg"+year+"/JG"+courseNumber+".doc"
    return link
```

*生成中文文件名*
通过必要信息生成下载后的文件名
```
def genfilename(courseName,teacher,courseNum):
    '''
    通过课程名，教师名，课程号生成下载后的文件名，按《out/学期/文件名.doc》存放
    '''
    str = "/out/" + courseNum[1:12] + "/" + courseName
    str = str + "_" + teacher + courseNum[courseNum.rfind("-"):] + "_" + courseNum[1:12] + ".doc"
    return str
```

*创建目录*
准备好用于存放下载文件的目录
```
def prepareFolder(file):
    '''
    这是检查目录是否存在，如果不存在就创建
    '''
    folder = os.path.dirname(file)
    if not os.path.exists(folder):
        print("创建目录：" + folder)
        os.makedirs(folder)
    
    return folder

```

*下载文件*
调用python库进行文件下载
```
def downfile(cname,teacher,cnum,url):
    '''
    下载文件
    '''
    ret = True
    response = http.request('GET', url)
    
    if response.status==200:
        filename = workFolder + genfilename(cname,teacher,cnum)
        prepareFolder(filename)
        with open(filename, 'wb') as f:
            f.write(response.data)
            
        print("保存文件：" + filename)
    else:
        print("未上传")
        ret = False
        
    response.release_conn()
    return ret
```
下载文件方法里，`url`是可以通过`cnum`计算出来的，因为在主程序结构中需要保存链接到Excel单元格，既然已经生成了，就作为参数传入好了，不用再重新生成一次，程序员的强迫症得到了体现:D ，美中不足是多了个参数

*改写主程序方法*
修改Excel行遍历，将定义好的各个方法放入到主程序结构中
```
for i in range(2,xlsSht.UsedRange.Rows.Count+1):
    courseName = xlsSht.Cells(i,5).Value
    teacher = xlsSht.Cells(i,6).Value
    courseNum = xlsSht.Cells(i,21).Value
    teacher = teacher.replace("/","_")
    print("\n"+str(i)+","+courseName+","+teacher+","+courseNum)
    
    link = genlink(courseNum)
    print("下载路径：" + link)
    stat = downfile(courseName,teacher,courseNum,link)
    if not stat:
        xlsSht.Cells(i,22).Value="未上传"
    else:
        xlsSht.Cells(i,22).Value=""
    
    xlsSht.Cells(i,23).Value=link
```

# 完工

程序写好了，放在python环境下跑，那节奏才叫个爽快，这才是现代化办公正确的打开方式嘛。需要完成这项任务的时候，就召唤出这个小精灵吧，剩下的五十多分钟就可以聊聊人生，谈谈理想啦（开个玩笑）。

不足之处，需要在用户电脑上安装python环境，需要用他们可能并不熟悉的方式召唤神龙，后面再考虑有没有使用起来更简单的方式，比如VBA...