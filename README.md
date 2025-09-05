# File Upload Attacks: 

## Main Headers : 
- File Name
- File Type
- Magic Number 
- File Content
- File Size

## Blacklist : 
- bypass via Case senistive
- (Apache - linux) --> .pht
- (IIS - Windows) --> .asa , .cer
- .emlto trigger stored xss in internet explorer only.

## Whitelist : 
- file.jpg.php
- Null byte injection : shell.php%001.jpg , shell.php\x00.jpg
- .svg --> xxe , xss
- ssrf in ffmpeg lib.
- ../../../index.php --> in File Name
- validating file content and let file name (شوف الكونتنت اللي متغيرش بعد الرفع بالهيكس وقارن الثابت وبدله ب الكود بتاعك)
- Image tragic attack --> image over {payload}.
- The file magic number is a png but the file is shell, this trick to bypass the content : GIF89a;<?php..>
- RCE via zip files
- OOB SQL in filename

## Automation :
- Upload scanner on burp.
