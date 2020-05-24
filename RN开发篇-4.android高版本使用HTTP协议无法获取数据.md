## Android 9.0及以上http请求限制

>问题：打包出来的release包在部分机型上无法获取到数据  

>原因：adnroid 9.0及以上版本默认禁止通过http进行网络请求

>解决方案：  
&emsp;&emsp;一、通过配置XML绕过HTTP限制  
&emsp;&emsp;在res文件夹下新建一个xml目录，新建如`netwrok_security_config.xml`文件，文件内容如下：  

    <?xml version="1.0" encoding="utf-8"?>  
    <network-security-config>  
      <base-config cleartextTrafficPermitted="true" />  
    </network-security-config>
&emsp;&emsp;然后在AndroidManifest.xml的Application中做如下配置：  

    android:networkSecurityConfig="@xml/network_security_config"

>&emsp;&emsp;二、通过在android/app/src/main/androidManifest.xml文件的Application中配置如下属性，指示应用程序允许使用明文网络流量（不安全，可能会导致网络攻击，如窃听传输的数据及修改数据）  

    android:usesCleartextTraffic="true"

>&emsp;&emsp;三、统一将http请求改为https（建议）

参考链接：  
&emsp;&emsp;1.https://www.jianshu.com/p/79f573bd0938  
&emsp;&emsp;2.https://blog.csdn.net/fancky58/article/details/90904632