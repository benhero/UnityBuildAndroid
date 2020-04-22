# UnityBuildAndroid
Unity自动化打包Android APK工程案例

## 零. 前言
最近在做Unity开发，有个任务是通过Jenkins实现自动化打包Unity游戏成Android APK的需求，所以在完成任务后，梳理成了一个最简单的教学，也把遇到的坑和大家分享。

若需要Jenkins打包，指需要通过Jenkins调用下面的命令即可实现基础需求。
## 一. 原理
打开Unity程序 → 打开指定工程 → 调用Unity的BuildPipeline类进行打包

## 二. 打包命令
#### Mac样例：
/Users/Ben/Application/Unity.app/Contents/MacOS/Unity -projectPath /Users/Ben/UnityBuildAndroid -executeMethod ProjectBuild.BuildForAndroid project-projectName -quit

#### Windows样例：
D:\Unity\Editor\Unity -projectPath D:\UnityBuildAndroid -executeMethod ProjectBuild.BuildForAndroid
project-projectName -quit

#### 格式说明：

![命令格式](https://github.com/benhero/UnityBuildAndroid/blob/master/Unity%E6%89%93%E5%8C%85APK.png?raw=true)


1. Unity编辑器路径：非UnityHub路径，可以在UnityHub → 安装 → 具体Unity版本 → 右上角更多信息 → 在Finder中显示(在资源管理器中显示) 可以找到对应Unity应用的路径
2. 编译的工程路径：需要打包APK的项目路径根目录
3. 执行的方法：编译APK运行的脚步
4. 参数(project-projectName)：样例中，projectName为工程名，用于命名APK。"project-projectName"整个不写，则会自动读取工程名。读者可以根据自己项目需求，传递需要的参数进入，然后自行解析处理。
5. -quit：在运行结束后关闭UnityHub，建议在前期未完全调通之前，不要添加该命令，否则打包失败报错时，无法通过Unity日志窗口查看具体原因。


## 三. 执行步骤
> 下面以全新Unity空工程来引入自动化打包

#### 1.引入打包工具类，将下面的代码拷贝到工程Assets/Editor目录下
```
using System.Collections;
using System.IO;
using UnityEditor;
using UnityEngine;
using System.Collections.Generic;
using System;

class ProjectBuild : Editor
{
    //在这里找出你当前工程所有的场景文件，假设你只想把部分的scene文件打包 那么这里可以写你的条件判断 总之返回一个字符串数组。
    static string[] GetBuildScenes()
    {
        List<string> names = new List<string>();
        foreach (EditorBuildSettingsScene e in EditorBuildSettings.scenes)
        {
            if (e == null)
                continue;
            if (e.enabled)
                names.Add(e.path);
        }
        return names.ToArray();
    }

    /// <summary>
    /// 自定义工程名："project-"作为工程名的前缀参数
    /// </summary>
    public static string projectName
    {
        get
        {
            foreach (string arg in System.Environment.GetCommandLineArgs())
            {
                if (arg.StartsWith("project"))
                {
                    return arg.Split("-"[0])[1];
                }
            }
            return Application.productName;
        }
    }

    /// <summary>
    /// 打包Android应用
    /// </summary>
    static void BuildForAndroid()
    {
        // 签名文件配置，若不配置，则使用Unity默认签名
        PlayerSettings.Android.keyaliasName = "BenheroGithub";
        PlayerSettings.Android.keyaliasPass = "BenheroGithub";
        PlayerSettings.Android.keystoreName = Application.dataPath.Replace("/Assets", "") + "/BenheroGithub.jks";
        PlayerSettings.Android.keystorePass = "BenheroGithub";

        // APK路径、名字配置
        string apkName = DateTime.Now.ToString("yyyyMMdd_HHmmss");
        string path = Application.dataPath.Replace("/Assets", "") + "/APK/" + projectName + "_" + apkName + ".apk";
        BuildPipeline.BuildPlayer(GetBuildScenes(), path, BuildTarget.Android, BuildOptions.None);
    }
}
```

这个类中3个方法比较清晰简单，读者可以根据需求进行拓展。

#### 2. 引入签名文件到工程的根目录下
#### 3. 配置Android应用包名
若不配置会报错：
UnityException: Package Name has not been set up correctly
Please set the Package Name in the Player Settings. The value must follow the convention

**解决方法：**
1. File/Build Settings切换Android平台
2. Player Settings → Other Settings → Identification → Package Name输入APK包名


#### 4. 在命令行窗口执行打包命令

#### 5. APK生成在工程根目录中的APK文件夹下

### 参考教程
[雨松MOMO](http://www.xuanyusong.com/archives/2748)
