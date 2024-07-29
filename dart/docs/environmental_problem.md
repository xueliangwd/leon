# 开发环境安装问题

- 1、devtools不能使用
Unhandled exception:
Invalid argument(s): A directory corresponding to fileSystemPath "/Users/yuxueliang/.pub-cache/hosted/pub.flutter-io.cn/devtools-2.9.2/build" could not be found#0   createStaticHandler (package:shelf_static/src/static_handler.dart:50:5)
#1   defaultHandler (package:devtools_server/src/external_handlers.dart:38:23)
<asynchronous suspension>
#2   serveDevTools (package:devtools_server/src/server.dart:215:15)
<asynchronous suspension>
#3   serveDevToolsWithArgs (package:devtools_server/src/server.dart:73:10)
<asynchronous suspension>
执行  dart pub global activate devtools -v 2.8.0 然后重启Android Studio

- 2、 flutter doctor --android-licenses    异常
盘符:\目录\tools\bin\sdkmanager  --update Exception in thread "main" java.lang.NoClassDefFoundError: javax/xml/bind/annotation/XmlSchema at com.android.repository.api.SchemaModule$SchemaModuleVersion.<init>(SchemaModule.java:156) at com.android.repository.api.SchemaModule.<init>(SchemaModule.java:75) at com.android.sdklib.repository.AndroidSdkHandler.<clinit>(AndroidSdkHandler.java:81) at com.android.sdklib.tool.sdkmanager.SdkManagerCli.main(SdkManagerCli.java:73) at com.android.sdklib.tool.sdkmanager.SdkManagerCli.main(SdkManagerCli.java:48) Caused by: java.lang.ClassNotFoundException: javax.xml.bind.annotation.XmlSchema at java.base/jdk.internal.loader.BuiltinClassLoader.loadClass(BuiltinClassLoader.java:583) at java.base/jdk.internal.loader.ClassLoaders$AppClassLoader.loadClass(ClassLoaders.java:178) at java.base/java.lang.ClassLoader.loadCla
解决方案：
1 先查看jdk版本 在cmd窗口中输入 java -version 查看你的jdk版本。
如果高于1.8就降低JDK 版本到 JDK 8。也可以不用卸载（删除），重新安装1.8的，然后环境变量的目录重新配置成1.8的目录就行了。
重新正确配置好jdk1.8 。 flutter doctor 继续愉快的进行后续操作吧。
- 3、MacOS切换JDK版本
https://juejin.cn/post/6871959224314757134
查看所有java版本
/usr/libexec/java_home -V
查看当前java版本
java -version
设全局版本
open ~/.bash_profile
# SWITCH TO JAVA VERSION 8
export JAVA_HOME=`/usr/libexec/java_home -v 1.8`
source .~/.bash_profile

- 4、Unable to find bundled Java version
android studio 提示 Unable to find bundled Java version
[!] Android Studio (version 2020.3)
  • Android Studio at /Applications/Android Studio Preview.app/Contents
  • Flutter plugin can be installed from:
   🔨 https://plugins.jetbrains.com/plugin/9212-flutter  • Dart plugin can be installed from:
   🔨 https://plugins.jetbrains.com/plugin/6351-dart  ✗ Unable to find bundled Java version.
  • Try updating or re-installing Android Studio.
处理方式：
    cd /Applications/Android\ Studio.app/Contents/jre
    ln -s ../jre jdk
    ln -s "/Library/Internet Plug-Ins/JavaAppletPlugin.plugin" jdk