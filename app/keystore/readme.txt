careland.keystore  密码是careland2012



MainApplication 文件里 初始化 ols库 要设置为正式版  initParam.isTestVersion 要为false；

Manifest 要改好导航签名 

测试版导航
 <meta-data
            android:name="com.cld.lbsapi.API_KEY"
            android:value="c814b2c470aqfiab534f5da6" />

正式版导航
 <meta-data
            android:name="com.cld.lbsapi.API_KEY"
            android:value="1e0816d9575akna3aa28370f" />


用要document文件里面libnavicm_misc_v1.0.so(正式版)