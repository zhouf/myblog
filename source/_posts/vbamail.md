---
title: 使用VBA进行邮件发送
date: 2018-02-24 10:05:20
tags:
---

在办公环境下面，VBA还是挺好玩的，当把一个繁琐的事情改造成一个自动化的过程的时候，就像又练成了一招功夫一般，或是又创造了一个小精灵。接下来又将创建一个小精灵，完成一个特别的任务

# 前言

对于经常用OFFICE的劳动人民来说，邮件合并是一个很能提高效率的功能，如工资条，成绩单等典型的应用。先说明一下今天的场景，有一个教学任务书，发给每个老师，因为每个老师的课程记录数不同，有的老师有一条记录，有的老师有多条记录，如果按每条记录给老师们发邮件，那有着多条记录的老师会收到多个邮件，这样做明显不太完美。网上也找了很多的办法，如在进行邮件合并时通过`NEXTIF`等宏命令来协助处理，一个老师如果有多条记录的情况下，对第一条记录可以很方便的设置，对后面的行操作起来也不是很友好，这个方案还有一个问题，就是任务记录少的老师会有多出来的空行，也不完美。

后面想了一下，邮件合并的核心思想就是把数据与文档进行关联，自动生成文档或邮件进行发送，现在数据在EXCEL文件里，目标就是邮件发送，那么可以通过数据生成HTML格式的邮件内容进行发送就好了，至于每个老师的任务记录个数，可以通过VBA程序来控制，这样就避免了上述的问题。想好了思路之后，开工:D

# 设计

为了用户操作方式和直观，在VBA中实现了一个界面，用户只需要选择发送数据的工作表和指定教师名所在的列就好，系统会根据不同的教师名发送邮件，每个老师的邮箱信息是单独保存到一个配置文件里的，如果有邮箱信息改变的情况，手动更改一下该文本文件即可，支持同一教师多个邮箱。这样设计省去了在工作表中提取邮箱信息的操作，通常需要发送数据的工作表都不包含邮箱信息的。让程序自动去匹配，没有的给出提示信息，省去了人工操作的步骤。

# 完成界面

在界面中提供邮件主题输入，除数据表格之外的正文部分，工作表选择，姓名列选择，以及操作按钮。程序中使用了两种邮件发送方式，本来是用于测试的，后来就保留了下来，做了一个选择框供用户选择，一种是调用Outlook发送，另一种是调用CDO组件发送。

界面设计在Visual Basic编辑器是选择`插入`->`用户窗体`即可，实现的界面效果如下图所示

{% asset_img mainform.png 操作界面 %}

# 程序模块部分

在程序模块中定义两个全局变量，一个是当前工作表，另一个是老师的邮箱数据，用一个字典保存
```
Public book As Workbook
Public mailDict As Object
```

在程序调用时，先进行数据初始化，从配置文件中加载邮箱数据，然后将当前工作簿的工作表数据传递到界面中，供用户选择
```
Sub 发送邮件()

    '这是一个发送邮件的操作
    Set book = Application.ActiveWorkbook

    '初始化数据
    Call initdata

    With MainForm.ComboSheet
        .Clear
        For Each sht In book.Worksheets
             'Debug.Print sht.Name
             .AddItem sht.Name
        Next
    End With

    MainForm.Show

End Sub
```
初始化数据的过程，从用户目录如`C:\Users\lenovo`下加载`mail.csv`配置文件，读入到字典变量中
```
Private Sub initdata()
    Dim txt As String, teacherName As String, email As String
    Dim cIndex As Integer

    Set mailDict = CreateObject("Scripting.Dictionary")

    '加载用户目录下的mail.csv文件
    Open Environ("UserProfile") & "\mails.csv" For Input As #1
    Do While Not EOF(1)
        Line Input #1, txt
        'Debug.Print txt
        cIndex = InStr(1, txt, ",")
        teacherName = Left(txt, cIndex - 1)
        email = Right(txt, Len(txt) - cIndex)

        mailDict.Add teacherName, email
    Loop
    Close #1
End Sub
```

在模块中还有两个发送邮件的方法，这个是在网上找到的，详细代码就不贴了
```
Public Function fSendEMailCDO(strTo As String, strSubject As String, strBody As String, Optional strAttachment As String = "", Optional strCC As String = "", Optional strBCC As String = "") As Object
....

Public Function SendMail(strTo As String, strSubject As String, strBody As String, Optional strAttachment As String = "", Optional strCC As String = "", Optional strBCC As String = "") As Integer
....
```
使用CDO方式需要在代码中配置邮件服务器信息，第二个SendMail方法直接调用配置好的Outlook发送邮件，如果调用Outlook发送邮件，需要引用`Microseft Outlook *.0 Object Library`库。接下来要处理的就是界面里的功能了

# 界面功能处理

选择工作表后，让用户选择教师姓名所在列
```
Private Sub ComboSheet_Change()
    Debug.Print ComboSheet.Text

    Debug.Print "index=" & ComboSheet.ListIndex
    Set sht = book.Sheets(ComboSheet.Text)

    Dim allcolumns As Integer

    allcolumns = sht.UsedRange.Columns.Count

    Dim v As String
    Dim l As Integer

    ComboColumn.Clear

    For r = 1 To allcolumns
        v = sht.Cells(1, r).Value
        ComboColumn.AddItem v
        l = l + Len(v)
    Next r

    If l = 0 Then
        MsgBox "所选择工作表" & ComboSheet.Text & "第一行未检测到标题数据"
    End If

End Sub
```
主要是点击发送按钮时的操作，在此操作中需要处理以下内容

* 检查界面数据的有效性
* 检查教师数据，是否能找到匹配的Email
* 对合并的教师如“教师A/教师B”需要进行拆分处理
* 通过数据生成HTML内容
* 进行邮件发送

```
'发送按钮
Private Sub SendBtn_Click()

    '检查页面是否填写完成
    If Len(TextSubject.Text) * Len(TextMail.Text) * Len(ComboSheet.Text) * Len(ComboColumn.Text) = 0 Then
        MsgBox "请填写完整"
        Exit Sub
    End If

    Set sht = book.Sheets(ComboSheet.Text)

    Dim allrows As Integer, allcolumns As Integer, teacherIndex As Integer
    Dim teacher As String

    allrows = sht.UsedRange.Rows.Count
    allcolumns = sht.UsedRange.Columns.Count
    teacherIndex = ComboColumn.ListIndex + 1

    Dim dic As Object
    Set dic = CreateObject("Scripting.Dictionary")

    For r = 2 To allrows
        teacher = sht.Cells(r, teacherIndex).Value
        Debug.Print teacher

        '对有/进行连接的多个名字进行分割

        For Each t In Split(teacher, "/")

            If Not dic.exists(t) And t <> "自动生成" Then
                dic.Add t, teacher
            End If
        Next

    Next

    Debug.Print "-------------------"

    For Each t In dic.keys
        Debug.Print t
        If Not mailDict.exists(t) Then
            MsgBox "系统中找不到" & t & "的邮箱信息，请检查是否拼写错误"
            Exit Sub
        End If
    Next

    Dim tbheader As String

    tbheader = genTableHeader(sht, allcolumns)

    '按表格中的教师获取数据
    For Each t In dic.keys
        Dim body As String

        body = "<html><h3>" & t & "老师，您好</h3>" & Chr(13) & Chr(10)
        body = body + "<h3>" & TextMail.Text & "</h3>" & Chr(13) & Chr(10)
        body = body + "<table border='1' style='border-collapse:collapse;' cellpadding='7'>" & Chr(13) & Chr(10)
        body = body + tbheader & Chr(13) & Chr(10)
        body = body + genTeacherData(sht, allcolumns, allrows, t, teacherIndex)
        body = body + "</table></html>"

        Dim address As String
        address = mailDict(t)

        Debug.Print "Mailto:" & address
        Debug.Print "subject:" & TextSubject.Text
        Debug.Print "MailBody:" & body

        If ckOutlook.Value = True Then
            a = SendMail(address, TextSubject.Text, body)
        Else
            Set a = fSendEMailCDO(address, TextSubject.Text, body)
            Debug.Print "发送结果：" & a
        End If

    Next

    MsgBox "完成发邮件" & dic.Count & "封", vbOKOnly, "发送完毕"
    Unload Me

End Sub
```

下面是几个被调用的函数，用表格的第一行数据生成HTML表头
```
Function genTableHeader(ByVal sht As Worksheet, allcols As Integer)
    '用第一行数据生成HTML表头
    Dim html As String

    html = html + "<tr>"
    For i = 1 To allcols
        html = html + "<th>" + sht.Cells(1, i).Value + "</th>"
    Next
    html = html + "</tr>"

    Debug.Print html

    genTableHeader = html
End Function
```

生成每位老师的HTML数据表信息
```
Function genTeacherData(ByVal sht As Worksheet, allcols As Integer, allrows As Integer, ByVal teacher As String, tIndex As Integer)
    '生成每位老师的数据字串
    Dim t As String, html As String, tdVal As String

    For r = 2 To allrows
        t = sht.Cells(r, tIndex)
        'Debug.Print "第" & r & "行教师：" & t

        If InStr(t, teacher) <> 0 Then
            '如果包含当前教师，则生成当前行数据
            Debug.Print "包含当前教师：" & teacher

            html = html + "<tr>"
            For i = 1 To allcols
                tdVal = sht.Cells(r, i).Value
                If tdVal = "" Then
                    html = html + "<td>&nbsp;</td>"
                Else
                    html = html + "<td>" + sht.Cells(r, i).Value + "</td>"
                End If
            Next
            html = html + "</tr>" & Chr(13) & Chr(10)
        End If
    Next

    genTeacherData = html

End Function
```
最后还有一个取消按钮的功能，就是关闭窗口，一条语句
```
Private Sub CancelBtn_Click()
    Unload Me
End Sub
```

# 小结

终于写完了，其实已经做完好久了，只是现在才把这篇文档整理出来，现在不管多少条记录，都可以通过这个功能发送邮件了，邮件合并的操作也省了，试了一下，效果还是很不错的，哈哈:D
