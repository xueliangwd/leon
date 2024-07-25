# iOS 安全

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [iOS 安全](#ios-安全)
  - [防越狱](#防越狱)
  - [防二次打包](#防二次打包)
  - [阻止动态调试](#阻止动态调试)
  - [网络安全](#网络安全)
  - [本地数据安全](#本地数据安全)
  - [代码反汇编](#代码反汇编)

<!-- /code_chunk_output -->


## 防越狱
1、检测是否被注入，阻止Cycript等的动态库注入。
2、在 Xcode 编译选项中 other linker flags 中添加 -Wl,- sectcreate,__RESTRICT,__restrict,/dev/null 标识。(注：这个方案只适合iOS10.0以下系统，而且个别系统打出来的包还会发生Crash，和Swift混编的项目时候可能会出问题，所以现在不建议用这种方式)
3、在关键业务上进行越狱检测，如果是越狱手机则进行提示或者直接退出程序，保证业务安全。
```objective-c
//是否越狱
+ (BOOL)isPrisonBreak
{
    if ([self isSimulator]) return NO;
    
    bool pb = NO;
    
    // 常见越狱文件
    NSArray *pathArray = @[
        @"/Applications/Cydia.app",
        @"/Library/MobileSubstrate/MobileSubstrate.dylib",
        @"/bin/bash",
        @"/usr/sbin/sshd",
        @"/etc/apt"
    ];
    for (int i = 0; i < pathArray.count; i++) {
        NSString *path = pathArray[i];
        if ([[NSFileManager defaultManager] fileExistsAtPath:path]) {
            pb = YES;
        }
    }
 
    // 读取系统所有的应用名称
    if ([[NSFileManager defaultManager] fileExistsAtPath:@"/User/Applications/"]){
        pb = YES;
    }
    
    if ([[UIApplication sharedApplication] canOpenURL:[NSURL URLWithString:@"cydia://"]]) {
        pb = YES;
    }
    
 
    struct stat stat_info;
    //使用stat系列函数检测Cydia等工具
    if (0 == stat("/Applications/Cydia.app", &stat_info)) {
        pb = YES;
    }
    
    // 读取环境变量
    char *checkInsertLib = getenv("DYLD_INSERT_LIBRARIES");
    if (checkInsertLib) {
        pb = YES;
    }
    
    return pb;
}
 
 
+ (BOOL)isSimulator {
#if TARGET_OS_SIMULATOR
    return YES;
#else
    return NO;
#endif
}
```
## 防二次打包
1、检测plist文件中是否有SignerIdentity值，SignerIdentity值只有ipa包被反编译后篡改二进制文件再次打包，才会有此值。(注：如果替换资源文件，比如图片、plist文件等是没有SignerIdentity这个值的。猜测只有改了二进制文件才会有此值(待验证) )
2、检测 cryptid 的值来检测二进制文件是否被篡改。网上说这也是一种解决方案，但是cryptid这个值好像在Mach-o中才有，目前还不知道如何获取该值。
3、IPA包上传到TestFlight或者App Store后，计算安装包中重要文件的MD5 值，服务器记录，在应用运行前首先将本地计算的 MD5 值和服务器记录的 MD5 值 进行对比，如不同，则退出应用。
```objective-c
+ (BOOL)isSecondIPA{
 
    //线上的包如果存在embedded文件，则认为是被二次打包了
    if ([[NSBundle mainBundle] pathForResource:@"embedded" ofType:@"mobileprovision"]) {
        return YES;
    } else {
        NSBundle *bundle = [NSBundle mainBundle];
        NSDictionary *info = [bundle infoDictionary];
        if ([info objectForKey: @"SignerIdentity"] != nil) {
            return YES;
        } else {
            return NO;
        }
    }
}
```
## 阻止动态调试
```objective-c
// 阻止 gdb/lldb 调试
// 调用 ptrace 设置参数 PT_DENY_ATTACH，如果有调试器依附，则会产生错误并退出
#import <dlfcn.h>
#import <sys/types.h>
 
typedef int (*ptrace_ptr_t)(int _request, pid_t _pid, caddr_t _addr, int _data);
#if !defined(PT_DENY_ATTACH)
#define PT_DENY_ATTACH 31
#endif
 
void anti_gdb_debug(void) {
    void *handle = dlopen(0, RTLD_GLOBAL | RTLD_NOW);
    ptrace_ptr_t ptrace_ptr = dlsym(handle, "ptrace");
    ptrace_ptr(PT_DENY_ATTACH, 0, 0, 0);
    dlclose(handle);
}
 
int main(int argc, char * argv[]) {
#ifndef DEBUG
    // 非 DEBUG 模式下禁止调试
    anti_gdb_debug();
#endif
    AppStartLaunchTime = CFAbsoluteTimeGetCurrent();
    NSString * appDelegateClassName;
    @autoreleasepool {
        // Setup code that might create autoreleased objects goes here.
        appDelegateClassName = NSStringFromClass([AppDelegate class]);
    }
    return UIApplicationMain(argc, argv, nil, appDelegateClassName);
}

```
## 网络安全
移动端与服务器通讯的网络传输过程中可能会经过不安全的中间节点，即使是HTTPS的加密通讯，Hacker也可能通过中间人攻击（Man-in-the-middle attack）来截取通讯内容，所以我们要对数据加密保护
- 关键数据(如登录密码、卡号、交易密码等)单独加密
- App内要对HTTPS证书做校验

## 本地数据安全
- 不要在iOS设备的Plist文件中保存敏感信息
- 数据库加密
- 键盘缓存关闭或标记为secure的字段、passwords字段
- 屏幕快照之前进行隐藏或模糊化处理
- 打包后去除日志

## 代码反汇编
- 增加代码混淆，可参照codeobscure
