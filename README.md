# XposedBridge
XposedBridge 不用重启即可生效，不用安装XposedInstaller
### 使用方法

1. 安装XBridgeIntaller

2. 修改XposedBridge.java类中MODULE_PACKAGE属性为自己模块类的类名，build应用，然后拷贝出apk文件，重命名为：XposedBridge.jar

`public static final String MODULE_PACKAGE = "xposed.sohu.com.xposeddemo";`

3. 通过XBridgeInstaller安装XPosed框架，安装成功后，替换system/framework/XposedBridge.jar文件。

4. 开发插件，更新代码后，重启应用就可以，不用再重启系统

### 原理
Zegote进程启动前会调用XposedBridge.java文件中的main方法，main方法中会加载hook插件，插件会被加载到Zegote进程，改变hook插件后要想生效，必须重新加载hook插件，所以必须重新开启Zegote进程，因此需要重新启动手机系统。要想改动插件代码不重新启动手机，不把hook插件load到Zegote进程，而是在自己的进程中load。
```
protected static void main(String[] args) {
		// Initialize the Xposed framework and modules
		try {
			if (!hadInitErrors()) {
				initXResources();

				SELinuxHelper.initOnce();
				SELinuxHelper.initForProcess(null);

				runtime = getRuntime();
				XPOSED_BRIDGE_VERSION = getXposedVersion();

				if (isZygote) {
					XposedInit.hookResources();
					XposedInit.initForZygote();
				}

//				XposedInit.loadModules();//1、不要Zegote进程中加载hook插件
			} else {
				Log.e(TAG, "Not initializing Xposed because of previous errors");
			}
		} catch (Throwable t) {
			Log.e(TAG, "Errors during Xposed initialization", t);
			disableHooks = true;
		}

		// Call the original startup code
		if (isZygote) {
			XposedHelpers.findAndHookMethod("com.android.internal.os.ZygoteConnection", BOOTCLASSLOADER, "handleChildProc",
					"com.android.internal.os.ZygoteConnection.Arguments",FileDescriptor[].class,FileDescriptor.class,
					PrintStream.class,new XC_MethodHook() {

						@Override
						protected void afterHookedMethod(MethodHookParam param) throws Throwable {
							// TODO Auto-generated method stub
							super.afterHookedMethod(param);
							String processName = (String) XposedHelpers.getObjectField(param.args[0], "niceName");
							String coperationAppName = MODULE_PACKAGE;
							Log.d(TAG,"only load xposed demo"+processName);
							if(processName != null){
								if(processName.startsWith(coperationAppName)){//2、只在自己插件进程中加载hook插件
									Log.d(TAG,"only load xposed demo:XX");
									XposedInit.loadModules();

								}
							}
						}

					});

			ZygoteInit.main(args);
		} else {
			RuntimeInit.main(args);
		}
```


