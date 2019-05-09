# XposedBridge
XposedBridge 不用重启即可生效，不用安装XposedInstaller
### 使用方法

1. 安装XBridgeIntaller

2. 修改XposedBridge.java类中MODULE_PACKAGE属性为自己模块类的类名，build应用，然后拷贝出apk文件，重命名为：XposedBridge.jar

`public static final String MODULE_PACKAGE = "xposed.sohu.com.xposeddemo";`

3. 通过XBridgeInstaller安装XPosed框架，安装成功后，替换system/framework/XposedBridge.jar文件。

4. 开发插件，更新代码后，重启应用就可以，不用再重启系统


