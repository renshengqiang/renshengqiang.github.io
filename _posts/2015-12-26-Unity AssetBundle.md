---
layout: post
author: tnqiang
titile: Unity AssetBundle
category: Unity
tag: Unity
---
#Unity AssetBundle

之前在一篇文章里面提到了传统的 Resources.Load 的资源加载方式中有两个问题：

1. Resources 目录是固定打包到安装包里面的，这个目录是一个只读目录，无法进行更新；
2. Resources.Load 函数只能从Resources 目录读取资源。

这两个问题促使我们如果想要去更新游戏中的资源的话，得去寻找 Resources 以外的其他方法，而这个方法就叫做 AssetBundle。

##AssetBundle是什么
Asset: 资源，Bundle: 包，所以 AssetBundle 合起来的意思就是资源包的意思。
在 Unity 中， AssetBundle 是存储资源的一种格式，这种格式可以存储 Unity 引擎所能识别的任意一种资源。换句话说，所有能够使用 Resources.Load 加载的资源都是可以使用 AssetBundle 去存储的。
既然有了这种存在形式，那么当前就会有相关的 API 来帮助我们从中读取资源并使用。

有了AssetBundle 我们可以做到：

1. 资源打包成 AssetBundle，支持动态更新；
2. 资源和安装包分离，按需下载。

一句话总结：有了 AssetBundle 后，程序和资源可以完全分离了。

##AssetBundle工作流
之前我们处理资源的时候就是直接丢到 Resources 目录就好，运行时直接就可以加载到。
有了 AssetBundle 之后，工作流变成下面这幅图所示的过程。
![](https://raw.githubusercontent.com/renshengqiang/renshengqiang.github.io/master/images/AssetBundle/workflow.png)

##创建 AssetBundle
![](https://raw.githubusercontent.com/renshengqiang/renshengqiang.github.io/master/images/AssetBundle/BuildAssetBundle.png)
这个 API 

- 把一个（mainAsset）或多个资源（assets）打包成一个资源包
- 打包输出为 pathName，就是我们需要的 assetbundle 文件
- 编译选项 assetBundleOptions 一般指定为CollectDependencies | 
CompleteAssets | DeterministicAssetBundle 即可
- 需要指定资源的平台（targetPlatform）

##加载 AssetBundle

1. 通过 WWW 加载
	- 缓存方式加载
	`	WWW www = WWW.LoadFromCacheOrDownload(url, version)`
	- 非缓存方式加载
   	`WWW www = new WWW(url)`
   	`AssetBundle ab = www.assetbundle`
2.  通过本地文件直接加载
AssetBundle AssetBundle.CreateFromFile(string path, int offset)
该函数**只支持非压缩**的AssetBundle，优点是同步调用速度快
3. 通过内存映像加载
AssetBundle AssetBundle.CreateFromMemory(byte[] binary)
这个函数通过两阶段来进行读取， File.Read 读到内存，然后使用这个函数读取 AssetBundle。
一般用于对assetbundle加密的时候使用。

一般使用 **非缓存方式加载** 加载，出于以下考虑：

- 打包的 AssetBundle 一般是压缩格式的，非压缩格式安装包太大；
- 缓存不需要 Unity 来帮我们做，每个手游有自己的版本更新系统；
- 一般项目中的资源是不需要进行加密的。

当然这个也是根据项目需要来定的，酌情选择。

##从 AssetBundle 加载 Object

{% highlight C# %}
Object AssetBundle.Load(
	string name,                                 /* 资源名，不包括路径和扩展名 */
	Type type);
 Object[] AssetBundle.LoadAll([Type type]);     /* 加载所有资源 */
 Object AssetBundle.mainAsset                    /* 功能糖函数，内部调用AssetBundle. Load */
{% endhighlight %}

简单纯粹的几个函数，不多做解释。
##场景打包和加载
除了 Resources 目录里面的资源外，场景也有动态更新的需求。

- 如果场景有误可以通过更新来更正
- 部分场景可以按需下载，减小安装包体积

打包函数：

{% highlight C# %}
public static string BuildStreamedSceneAssetBundle(
	string[] levels, 			    /* 打包的关卡id */
	string locationPath, 		    /* 打包的AB名 */
	BuildTarget target, 		    /* 平台 */
	BuildOptions options = 0);		/* 编译选项 */

{% endhighlight %}

加载和释放过程：
![](https://raw.githubusercontent.com/renshengqiang/renshengqiang.github.io/master/images/AssetBundle/sceneAssetBundle.png)


##AssetBundle 资源释放

- 释放 AssetBundle 本身：AssetBundle.Unload(false)
- 释放 AssetBundle 和从 AssetBundle 中加载出来的 GameObject: AssetBundle.Unload(true)

掌握了上面这些我们就可以使用 AssetBundle 加载资源了，后面介绍一些细节。

##AssetBundle 内存结构
![](https://raw.githubusercontent.com/renshengqiang/renshengqiang.github.io/master/images/AssetBundle/assetbundleMemoryStructure.gif)
这个动画大概介绍了AssetBundle 在加载和释放时候的内存结构。
注意点：
AssetBundle本身的资源我们统称为 webstream，这部分内存和加载出来的 GameObject 需要的资源内存量相当（webstream:Object ~= 1.25：1），因此加载完 Object 后需要及时释放。

##打包参数详解

|  参数                         | 作用          |
| -------------                |:-------------:|
| CollectDependencies          | 将该资源依赖的所有资源都打包进来 |
| CompleteAssets               | 将打包资源的完整资源都打包进来      |
| DisableWriteTypeTree         | 在资源包不包含类型信息(此标志只影响WebPlayer平台，Standalone和Mobile不包含类型信息)|
| DeterministicAssetBundle     | 保证资源多次打包md5值是相同的      |
| UncompressedAssetBundle      | 压缩与否      |

- 将资源的依赖打包进来需要指定 CollectDependencies
- DeterministicAssetBundle选项对于版本运营非常有用，可以保证同一个资源在两次更新间如果没有发生变化打包结果完全相同。原因在于打包 assetbundle 需要计算 hash 值，而 Unity 在选择 hash 值的时候有自己的选择性，如果不指定则每次的hash值都是不同的
- 压缩比约为0.25，一般建议是压缩

##平台兼容性

|     \     |Standalone|Webplayer|iOS| Android|
| ----------|:--------:| :------:|:-:| ------:|
| Editor    | Y        | Y    | Y     | Y     |
| Standalone| Y        | Y    |       |       |
| Webplayer | Y        | Y    |       |       |
| iOS       |          |      | Y     |       |
| Android   |          |      |       | Y     |

- 各个平台基本不兼容
- Editor 下可以使用四个平台的AssetBundle，方便调试

##资源打包冗余问题
![](https://raw.githubusercontent.com/renshengqiang/renshengqiang.github.io/master/images/AssetBundle/RedundancyBuild.png)

a.prefab 和 b.prefab 需要打包，不做任何处理打包他们结果是：d.png 和 e.ttf 打包两份进去，浪费内外存。

##拆分依赖，单独打包
![](https://raw.githubusercontent.com/renshengqiang/renshengqiang.github.io/master/images/AssetBundle/parseDependency.png)
打包思路：

- 共享资源单独打包，不留冗余；
- 能够合并打包的尽量合并打包。

缺点：
加载一个资源的时候需要加载的资源太多，IO频繁，场景加载慢

##允许一定冗余打包
![](https://raw.githubusercontent.com/renshengqiang/renshengqiang.github.io/master/images/AssetBundle/allowSomeRedundancy.png)
打包思路：
- 资源分成两种类型：基础模块资源（字体，Shader等），游戏模块资源（一个UI界面，一个主角，一把武器等）
- 资源依赖只有一种：游戏模块依赖基础模块；
- 基础模块单独打包；
- 每个模块资源单独打包。

##依赖如何打包
Unity提供了两个函数帮助我们来将依赖关系编译进 AssetBundle 中：
```
BuildPipeline.PushAssetDependencies()
BuildPipeline.PopAssetDependencies()
```
执行 Push 函数后，然后 Build A 资源，此时A资源会作为一个共享资源；如果B依赖于A，则 Build B 的时候 B 中便没有了 A。pop执行相反操作，将上一个共享资源从栈中去除。
例如如下的依赖关系，打包结果为：
![](https://raw.githubusercontent.com/renshengqiang/renshengqiang.github.io/master/images/AssetBundle/dep.png)
打包顺序为：

{% highlight C# %}
PushAssetDependencies();
BuildAssetBundle(D);
PushAssetDependencies();
BuildAssetBundle(B);
PushAssetDependencies();
BuildAssetBundle(C);
PushAssetDependencies();
BuildAssetBundle(A);
PopAssetDependencies();
PopAssetDependencies();
PopAssetDependencies();
PopAssetDependencies();

{% endhighlight %}

##依赖如何加载和释放
加载的时候只需要保证加载的所有的依赖都加载到内存中即可，不需要考虑时序；
释放的时候更加不需要考虑时序，直接释放即可。

##Prefs：
1. [【进阶】Unity资源解决方案之AssetBundle](http://gad.qq.com/college/articledetail/43)
2. [AssetBundle内存关系图](http://blog.sina.com.cn/s/blog_80cc3d870101kxzb.html)
3. [AssetBundle 5.0](http://www.jianshu.com/p/74f48982af43)
4. [Unite 2014AssetBundle 5.0](https://www.youtube.com/watch?v=gVUgF2ZHveo)
