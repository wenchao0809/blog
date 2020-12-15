### 1. ZIP
`zip`格式已成为压缩文件的标准选择，而且它在`windows`上也能使用。经常用`zip`格式压缩那些需要共享给`windows`用户的文件。
如果只是共享给`Linux`用户或者`Mac`用户，偏向于选择`tar.gz`格式。

~~~bash
# 压缩目录
 zip -r archive_name.zip directory_to_compress
 # 解压一个zip文档
 unzip archive_name.zip
~~~
### 2. TAR
tar是在Linux中使用得非常广泛的文档打包格式。它的好处就是它只消耗非常少的CPU以及时间去打包文件，注意它仅仅只是一个打包工具，并不负责压缩。

~~~bash
# 打包
 tar -cvf archive_name.tar directory_to_compress
 # 解包
 tar -xvf archive_name.tar.gz
~~~
### 3. TAR.GZ
这种格式是我使用得最多的压缩格式。它在压缩时不会占用太多CPU，而且可以得到一个非常理想的压缩率。
使用下面这种格式去压缩一个目录：

~~~bash
 # 压缩 
 tar -zcvf archive_name.tar.gz directory_to_compress
 # 解压
 tar -zxvf archive_name.tar.gz -C /tmp/extract_here/
~~~
### 4. TAR.BZ2
这种压缩格式是我们提到的所有方式中压缩率最好的。当然，这也就意味着，它比前面的方式要占用更多的CPU与时间。

使用tar.bz2进行压缩：

~~~bash
# 压缩
tar -jcvf archive_name.tar.bz2 directory_to_compress
# 解压
tar -jxvf archive_name.tar.bz2 -C /tmp/extract_here/
~~~