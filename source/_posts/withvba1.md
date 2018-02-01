---
title: 改用VBA完成自动化任务
date: 2018-02-01 10:09:09
tags:
---
之前写了一篇用python来完成自动化任务的文档，后面试着用VBA来改写了一遍，VB这样的语法许久没用了，上学的时候考试完就没太用过，写过一些VBA，也不经常用，所以改起来还是没有那么的流畅。想到用VBA来改写是希望让用户操作起来更为方便一点，添加一个自定义工具，点个按钮就召唤出来了，这样的操作应该没有太多的门槛吧。

# 前言

程序的功能和思路基本上和上一篇文档一样，只是添加了一些处理让程序适应性更强，比如可以适应不同的列号，也就是说关键数据的列位置可以不固定，表格前面允许有空行，让程序去略过。大致的步骤也是遍历表格，计算出数据，进行下载，保存，更新表格数据，提示是否需要查看下载的目录，方便用户下载完成后直接打开，不用再通过路径去找下载文件。

# 程序部分

首先是主体程序结构，也是VBA调用入口，本来计划是用英文字符来命名的，从用户的角度出发，还是就用中文了
```
Sub 检查教纲提交()
    
    Set book = Application.ActiveWorkbook
    Set sht = book.ActiveSheet
    
    Dim allrows As Integer, allcolumns As Integer
    Dim downcnt As Integer, errcnt As Integer
    Dim course As String, teacher As String, courseNumber As String, folder As String
    Dim courseIndex As Integer, teacherIndex As Integer, courseNumberIndex As Integer, datarow As Integer
    Dim check As Boolean
    
    allrows = sht.UsedRange.Rows.Count
    allcolumns = sht.UsedRange.Columns.Count
    'rows = UsedRange.rows.Count
    'allrows = 5
    
    '判断工作表并准备数据列
    check = prepare(allrows, allcolumns, courseIndex, teacherIndex, courseNumberIndex, datarow)
    
    If check = False Then
        '如果工作表不符合条件
        MsgBox "当前工作表中不包含“课程名称”“教师姓名”“选课课号”等数据列，无法完成当前操作"
        Exit Sub
    End If
    
    Application.ScreenUpdating = False
    '准备目录
    
    folder = book.Path & "\out"
    checkAndCreateFolder (folder)
    
    '循环处理
    For i = datarow To allrows
        course = Cells(i, courseIndex).Value
        teacher = Cells(i, teacherIndex).Value
        courseNumber = Cells(i, courseNumberIndex).Value
        
        'If course = "课程名称" Then
        '    Cells(i, 22).Value = "是否上传"
        '    Cells(i, 23).Value = "链接地址"
        'End If
        
        If Len(course) > 0 And course <> "课程名称" Then
            Dim link As String
            
            If InStr(1, courseNumber, "000000") = 0 Then
                '排除自动生成
                link = getlink(courseNumber)
                teacher = Replace(teacher, "/", "-")
                
                Debug.Print course & teacher & courseNumber
                Debug.Print link
                
                Cells(i, courseNumberIndex + 2).Value = link
                ret = downAndSave(course, teacher, courseNumber, link, folder)
                Debug.Print "下载操作结果：" & ret
                
                If ret = False Then
                    '下载失败
                    Cells(i, courseNumberIndex + 1).Value = "未上传"
                    errcnt = errcnt + 1
                Else
                    downcnt = downcnt + 1
                End If

            End If

            Debug.Print "----------------------------------------"
        End If
        
    Next
    
    Application.ScreenUpdating = True
    
    Dim msg As String
    msg = "执行完毕，共下载文件" & downcnt & "个，" & errcnt & "个未上传" & Chr(13) & "文件保存到：" & folder
    
    Debug.Print msg
    'MsgBox msg

    openfolder book.Path & "\out", msg
    '------------------------------------------------------------
    
End Sub
```

其中有很多功能的拆分，用函数和过程实现的，接下来就一一解释一下（排名不分先后）

```
Function prepare(ByVal allrows As Integer, ByVal allcols As Integer, ByRef courseCol As Integer, ByRef teacherCol As Integer, ByRef cnumCol As Integer, ByRef datarow As Integer)
    '准备运行程序，查找当前表格中的符合条件的数据列，如果当前表格不符合，则返回False
    '如果符合条件，则获取“课程名称”“教师姓名”“选课课号”所在列序号，并标记下一行为数据行
    
    Dim checkok As Boolean
    
    Debug.Print allrows, allcols, courseCol, teacherCol, cnumCol, datarow
    
    For r = 1 To allrows
        For c = 1 To allcols
            
            Debug.Print Cells(r, c).Value
            
            If Cells(r, c).Value = "课程名称" Then
                courseCol = c
            ElseIf Cells(r, c).Value = "教师姓名" Then
                teacherCol = c
            ElseIf Cells(r, c).Value = "选课课号" Then
                cnumCol = c
            End If
        
            If courseCol * teacherCol * cnumCol <> 0 Then
                '找到了所需要的数据列
                checkok = True
                datarow = r
                Exit For
            End If
            
        Next c
        
        If checkok = True Then Exit For
        
    Next r
    
    If checkok = True Then
        '如果符合操作条件，添加新的数据列
        Cells(datarow, cnumCol + 1).Select
        Selection.EntireColumn.Insert , CopyOrigin:=xlFormatFromLeftOrAbove
        Selection.EntireColumn.Insert , CopyOrigin:=xlFormatFromLeftOrAbove
        Cells(datarow, cnumCol + 1).Value = "是否上传"
        Cells(datarow, cnumCol + 2).Value = "链接地址"
        datarow = datarow + 1
    End If
    
    Debug.Print allrows, allcols, courseCol, teacherCol, cnumCol, datarow
    
    prepare = checkok

End Function
```
在此方法中检查数据表是否包含操作所需的数据列，如果满足条件，添加操作记录列“是否上传”，“链接地址”，需要考虑带回数据，在参数里添加了很多的引用。接下来是通过课程号生成下载链接地址的方法
```
Function getlink(curNumber As String)
    '通过课程号生成下载链接地址
    Dim year As String
    year = Mid(curNumber, 4, 2) & Mid(curNumber, 9, 2)
    getlink = "http://192.168.12.39/kcjg" & year & "/JG" & curNumber & ".doc"

End Function
```
这个方法比较简单，主要是对字串的处理。接下来是下载文件的方法
```
Function downAndSave(ByVal course As String, ByVal teacher As String, ByVal cnumber As String, ByVal url As String, ByVal folder As String)
' 下载并保存文件
    Dim fileName As String
    On Error GoTo MyErr
    
    folder = folder & "/" & Mid(cnumber, 2, 11)
    checkAndCreateFolder (folder)
    
    fileName = folder & "/" & course & "_" & teacher & "_" & Right(cnumber, Len(cnumber) - InStrRev(cnumber, "-")) & ".doc"
    'fileName = book.Path & fileName
    Debug.Print fileName
    
    Dim xmlhttp As Object
    Set xmlhttp = CreateObject("Microsoft.XMLHTTP")
    xmlhttp.Open "GET", url, False
    xmlhttp.Send

    Do While xmlhttp.readystate <> 4 '等待完成
        DoEvents
    Loop
    
    If xmlhttp.Status = 404 Then
        '如果资源不存在
        GoTo MyErr
    End If
    
    Set outfile = CreateObject("ADODB.Stream")
    outfile.Type = 1
    outfile.Open
    outfile.write xmlhttp.Responsebody
    outfile.savetofile fileName, 2    '本地保存文件名
    outfile.Close

    downAndSave = True
    Exit Function
    
MyErr:
    Debug.Print "下载出错"
    downAndSave = False
    
End Function
```
传入操作所需要参数，创建相应的保存目录。这个方法如果服务器故障无响应，程序会有长时间的停顿，目前暂未处理这种情况，未有添加url可访问检测。下面是处理目录的操作，检查目录是否存在。系统会在当前xls文件所在目录下创建“out”目录，在其中创建“学期”目录，都是由此方法完成
```
Function checkAndCreateFolder(ByVal folder As String)
    '检查目录是否存在，如果不存在，则创建
    On Error GoTo er
    
    If Dir(folder, vbDirectory) = Empty Then
        MkDir folder
    End If
    
    Debug.Print "创建目录" & folder
    
    Exit Function
er:
    Debug.Print folder & "目录已经存在"
End Function
```
程序执行完成后，下载完所有的文档，并在表格中做好标记，则提示是否需要打开目录，打开目录调用如下过程
```
Private Sub openfolder(ByVal folder As String, msg As String)
    '是否打开目录
    ans = MsgBox(Prompt:=msg, Buttons:=vbYesNo, Title:="是否查看目录")
    If ans = 6 Then
        '选择yes
        mOpen = Shell("Explorer.exe " & folder, vbNormalFocus)
    End If

End Sub
```

# 开发小记

当前程序功能简单，未做界面，目前可以通过宏直接调用，需要通过设置打开【开发工具】，Function和Private的过程不会出现在调用列表中，所以剩下的那个调用过程就用了中文名。

另一种方法是不用打开【开发工具】选项卡，在【选项】中【自定义功能区】里将宏命令添加到功能区中，下次直接点击功能区按钮执行。

还需要注意一个问题就是宏保存的位置，通常在XLS文件中写VBA多是保存在当前文件中，换一个文件可能就无法调用所写的功能了，如果希望其它的XLS文件也能调用到，建议把宏写在【个人宏工作簿】如下图
{% asset_img pic1.png 选择宏保存位置 %}

可以先录制一个简单的宏，然后编辑，在工程视图里就可以看到`PERSONAL.XLSB`的模块了，如下图
{% asset_img pic2.png 个人工作簿宏模块 %}
保存在当前模块下的宏可以用在所有打开的文档中，我们可以为不同的功能创建不同的模块，上面的代码可以放入此模块中，这样调整后，代码会有所变化，之前在当前工作簿中获取活动工作表，可以用
```
Set sht = ThisWorkbook.ActiveSheet
```
如果宏放在`PERSONAL.XLSB`中，上面的代码获取到的活动工作表就是`PERSONAL.XLSB`中工作表，如果要获取当前打开文档的活动工作表，需要改为如下的写法
```
Set book = Application.ActiveWorkbook
Set sht = book.ActiveSheet
```

这次的内容就到此吧，用VBA还算挺好玩的，几年前帮朋友做了一个数据统计整理的VBA，用起来挺爽快的，只没有写文档。这次也帮忙写了个小程序，点击一个按钮就完成了工作，就像是召唤出了一个小精灵，可爱的小精灵。要解决工作中重复繁琐的问题，VBA还是可以学学的嘛