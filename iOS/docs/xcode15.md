# 升级Xcode 15后项目错误处理

- 1、type argument ‘nw_proxy_config_t‘ is neither an Objective-C object nor a block type
```ruby
在podfile最后添加
post_integrate do |installer|
  compiler_flags_key = 'COMPILER_FLAGS'
  project_path = 'Pods/Pods.xcodeproj'

  project = Xcodeproj::Project.open(project_path)
  project.targets.each do |target|
    target.build_phases.each do |build_phase|
      if build_phase.is_a?(Xcodeproj::Project::Object::PBXSourcesBuildPhase)
        build_phase.files.each do |file|
          if !file.settings.nil? && file.settings.key?(compiler_flags_key)
            compiler_flags = file.settings[compiler_flags_key]
            file.settings[compiler_flags_key] = compiler_flags.gsub(/-DOS_OBJECT_USE_OBJC=0\s*/, '')
          end
        end
      end
    end
  end
  project.save()
end

```
参考文档：
https://blog.csdn.net/crasowas/article/details/133190660
https://stackoverflow.com/questions/77189557/type-argument-nw-proxy-config-t-is-neither-an-objective-c-object-nor-a-block-t


- 2、Xcode15.0打包Command PhaseScriptExecution failed with a nonzero exit code在项目中搜索

ios/Pods/Target Support Files/Pods-Runner/Pods-Runner-frameworks.sh
1
然后把
source="$(readlink "${source}")"
替换成
source="$(readlink -f "${source}")"
————————————————
原文链接：https://blog.csdn.net/CSND7997/article/details/133947867

https://cloud.tencent.com/developer/article/2255102
https://segmentfault.com/q/1010000043614798

- 3.真机运行报错Thread 1 Queue : com.apple.main-thread (serial)
#0    0x0000000000000000 in 0x00000000 ()
运行崩溃卡在c++错误 Factroy_ResLoader::get_inst
在build setting -> Other Linker Flags 添加 -Wl,-ld_classic （该参数在xcode14会导致编译失败，低版本需去掉）

- 4.模拟器运行
[Client] Synchronous remote object proxy returned error:终端运行
xcrun simctl spawn booted logconfig--mode "level:off"  --subsystem com.apple.CoreTelephony

https://blog.csdn.net/darongzi1314/article/details/88639906