app签名，相当于是app在Anndroid系统上的一个认证，Android系统要求每一个Android应用程序必须要经过数字签名才能够安装到系统中，也就是说如果一个Android应用程序没有经过数字签名，是没有办法安装到系统中的！Android通过数字签名来标识应用程序的作者和在应用程序之间建立信任关系，不是用来决定最终用户可以安装哪些应用程序。这个数字签名由应用程序的作者完成，并不需要权威的数字证书签名机构认证，它只是用来让应用程序包自我认证的。应用市场上APP签名不允许相同，也不会相同，但允许有相同的包名，相同签名的APP高版本可以覆盖低版本。

在开发过程中，如果没有手动给app添加签名，ADT会自动的使用debug密钥为应用程序签，debug密钥是一个名为debug.keystore的文件，它的位置在：C:/${user}/.android/debug.keystore 。也就是说，如果想拥有自己的签名，而不是让ADT使用自动生成的debug.keystore签名的话，需要有一个属于自己的密钥文件（*.keystore）。

> 默认的 debug.keystore 位置如下：
>
> //https:////upload-images.jianshu.io/upload_images/3744244-8e5a04cf11ddd80e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp
>
> ![img](https:////upload-images.jianshu.io/upload_images/3744244-8e5a04cf11ddd80e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)
>
> image.png

**以下是在命令行下，ionic 安卓app签名步骤**

## 准备工作

- keytool：该工具位于jdk安装路径的bin目录下；
- jarsigner：该工具位于jdk安装路径的bin目录下；
- zipalign：该工具位于Android-sdk-windows/tools/目录下；

> keytool和jarsigner两个工具是jdk自带的，也就意味着生成数字证书和文件签名不是Android的专利，jarsigner主要是用来给jar文件签名的。配置了JAVA环境变量，keytool和jarsigner可以直接在命令行下使用。zipalign 可能新老版本不太相同，可以在ANDROID_HOME下全局搜索zipalign.exe文件，以下是我电脑上的文件路径：
>
> 
>
> ![img](https:////upload-images.jianshu.io/upload_images/3744244-1d7cbd51cdf0936e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)
>
> image.png



## 生成未经签名的apk文件

Android app 的打包分为 debug 和 release 两种，后者是用来发布到应用商店的版本。在开发ionix项目是，使用默认命令(ionic cordova build android)打包出来的是debug apk文件。要打包 release 版本的apk文件，只需要在后面加一个 --release 参数即可：

```
ionic cordova build android --release
执行该命令后，会在   ionic项目根目录\platforms\android\build\outputs\apk    目录 
下生成一个 “android-release-unsigned.apk” 文件，这个apk文件就是 没有使用默认签名的 文件。
```



## 签名

###### 使用keytool 生成数字证书

```
keytool -genkey -v -keystore spilledyear.keystore -alias spilledyear.keystore -keyalg RSA -validity 36500

keytool是工具名称  
-genkey意味着执行的是生成数字证书操作 
-v表示将生成证书的详细信息打印出来，显示在dos窗口中  
-keystore spilledyear.keystore 表示生成的数字证书的文件名为“ spilledyear.keystore”(spilledyear可以取自己的名字) 
-alias spilledyear.keystore 表示证书的别名为“spilledyear.keystore”，当然可以不和上面的文件名一样 
-keyalg RSA 表示生成密钥文件所采用的算法为RSA 
-validity 36500 表示该数字证书的有效期为36500天，意味着36500天之后该证书将失效 
```

> 在执行上面的命令生成数字证书文件时，会提示你输入一些信息，包括证书的密码，如图所示：
>
> 
>
> ![img](https:////upload-images.jianshu.io/upload_images/3744244-5d03f02c8142ceb0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)
>
> keystore.png

###### 使用jarsigner为app签名

```
jarsigner -verbose -keystore spilledyear.keystore -signedjar zmjj.apk android-release-unsigned.apk spilledyear.keystore

jarsigner是工具名称 
-verbose表示将签名过程中的详细信息打印出来，显示在dos窗口中
-keystore spilledyear.keystore 表示签名所使用的数字证书所在位置，没有写路径表示在当前目录下
-signedjar zmjj.apk android-release-unsigned.apk 表示给android-release-unsigned.apk文件签名，签名后的文件名称为zmjj.apk 
spilledyear.keystore 表示证书的别名，对应于生成数字证书时-alias参数后面的名称
```

> 运行该命令，结果如下图所示：
>
> 
>
> ![img](https:////upload-images.jianshu.io/upload_images/3744244-45a64f2a548092d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)
>
> jarsigner01.png
>
> 
>
> ![img](https:////upload-images.jianshu.io/upload_images/3744244-0bac020e0b048c93.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)
>
> jarsigner02.png
>
> 
>
> ![img](https:////upload-images.jianshu.io/upload_images/3744244-bf63f028b8d68bdc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)
>
> jarsigner03.png

###### 使用zipalign优化已签名的apk

此步骤时非必需操作，但是建议这么做。

```
zipalign -v 4 zmjj.apk zmjj_aligned.apk
zipalign是工具名称 
-v表示在DOS窗口打印出详细的优化信息 
zmjj.apk zmjj_aligned.apk 表示对已签名文件 zmjj.apk进行优化，优化后的文件名为zmjj_aligned.apk
```

> 执行以上命令，结果如下图所示：
>
> 
>
> ![img](https:////upload-images.jianshu.io/upload_images/3744244-10f161f94796c754.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)
>
> image.png
>
> 此时在目录下又多生成了一个文件，zmjj_aligned.apk  ，也就是被压缩优化过的apk文件：
>
> 
>
> ![img](https:////upload-images.jianshu.io/upload_images/3744244-db8892797e3f88a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)
>
> image.png

如果以前的程序是采用默认签名的方式(即debug签名)，一旦换了新的签名，应用将不能覆盖安装，必须将原先的程序卸载掉，才能安装上。因为程序覆盖安装主要检查两点：

- 两个程序的入口Activity是否相同。两个程序如果包名不一样，即使其它所有代码完全一样，也不会被视为同一个程序的不同版本；
- 两个程序所采用的签名是否相同。如果两个程序所采用的签名不同，即使包名相同，也不会被视为同一个程序的不同版本，不能覆盖安装。

另外，可能有人可能会认为反正debug签名的应用程序也能安装使用，那也没有必要自己签名了。千万不要这样想，debug签名的应用程序有这样两个限制，或者说风险：

- debug签名的应用程序不能在Android 应用商店上架销售，它会强制你使用自己的签名。
- debug.keystore在不同的机器上所生成的可能都不一样，就意味着如果换了机器对app打包升级，那么将会出现上面那种程序不能覆盖安装的问题。

作者：spilledyear

链接：https://www.jianshu.com/p/26166279413b

来源：简书

简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。



![1556590906367](C:\Users\thinkpad\AppData\Roaming\Typora\typora-user-images\1556590906367.png)





打包出错可用android studio 打包

![1556594876753](C:\Users\thinkpad\AppData\Roaming\Typora\typora-user-images\1556594876753.png)