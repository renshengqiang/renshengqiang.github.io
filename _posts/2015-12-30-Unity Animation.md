---
layout: post
author: tnqiang
titile: Unity 动画基础
category: Unity
tag: Unity
---
##基本概念

###Animation
Animation是一种关键帧信息的集合。
例如 Tranform 动画的一个例子如下：

{% highlight C# %}
{
	{Keyframe1(time1,  (node1.translation1, node2.translation1 ))},
	{Keyframe2(time2,  (node1.translation2, node2.translation2 ))},
	...
}

{% endhighlight %}

其中每个关键帧记录了当前关键帧的**动画时间**和动画涉及到的**每个节点的当前帧位移**，动画的过程中会根据当前时间找到每个节点的偏移，对 GameObject 的 Transform 进行改变。

例如当前的动画时间是time, r = (time - time1)/(time2 - time1)，则插值结果：

- node1 节点的动画偏移：(1-r)*node1.translation1 + r*node1.translation2;

- node2 节点的动画偏移：(1-r)*node2.translation1 + r*node2.translation2。

整个动画执行的过程就是根据当前的动画和动画时间进行插值的过程。Unity中使用 .anim 文件来保存这些动画关键帧信息。

###动画种类

- 节点动画
这种类型的动画比较常见于 UI 动画中，它的特点是：动画的关键帧直接控制的是我们想要动画的对象。
- 骨骼（蒙皮）动画
这种动画比较常见于三维角色动画，它的特点是：动画的关键帧控制的是骨骼节点的偏移等，蒙皮节点受骨骼节点影响而进行动画。因此动画是间接控制我们想要的对象的。
	
Unity 中进行了一层封装，用户看到的只是 Animation，当然我们也不需要关心。
	
###Animation Clip
Unity 中的一个概念，表示动画当中的一个片段。当美术将所有的动画都做在一个anim 中的时候，我们需要对这个 anim 进行切分成一个个的 Animation Clip 用于动画控制。
	
###Animation Component
![](https://raw.githubusercontent.com/renshengqiang/renshengqiang.github.io/master/images/UnityAnimation/animationComponent.png)
Animation 组件是用来对 Unity 中的 Animation Clip 进行简单封装的一个功能组件，用于播放动画。当设置了 Play Autiomatically 选项后，Animation 字段对应的动画片段会自动播放。

###Animator
Animator 是 Unity 对 Animation Clip 进行的较高层级封装的一个功能组件，引入状态机的概念来进行动画控制，便于具有复杂功能的动画切换的实现。
![](https://raw.githubusercontent.com/renshengqiang/renshengqiang.github.io/master/images/UnityAnimation/animationStateMachine.png)
Animator 中将每个状态抽象成一个 Animation State，一个 Animation State 是一个<AnimationClip, 转换条件1， 转换条件2 ...>的一个多元组。
![](https://raw.githubusercontent.com/renshengqiang/renshengqiang.github.io/master/images/UnityAnimation/animationState.png)
同时 Unity 为了方便使用加入了 Layer 和 Default State 的概念，防止状态机过于复杂。
Animator 的状态切换API叫做：Animator.SetXXX("param", value)
![](https://raw.githubusercontent.com/renshengqiang/renshengqiang.github.io/master/images/UnityAnimation/animator.png)

###Avator
游戏中人形结构的游戏体特别常见，因此 Unity 对人形的骨骼结构定义了一套骨骼结构。
当模型导入的时候，Unity 会尝试将模型中的骨骼节点和 avatar 节点进行一个对应，这个对应关系就是一个 avatar。有了这个 avatar，其他模型的动作也可以用到这个模型中。

##Unity中动画处理的工作流
这里我们只讨论角色骨骼动画的工作流，UI节点动画一般不需要状态机控制，美术给到 Animation 直接挂接到UI上面会自动播放，无需程序控制。

1. 美术给到的资源：模型和动画；
2. 根据美术给的模型创建 prefab；
3. 新建一个状态机（animation controller），设定好角色需要的几个动作然后设置他们之间的转换条件；
4. prefab上面挂接一个 Animator 组件，其中的 Animation Controller 使用之前创建的状态机即可；
5. 程序中后去这个 Animator 组件，然后使用 SetXXX 方法对状态机进行控制，控制角色的工作。

##动画预览

1. 先选中骨骼动画文件；
2. 将模型拖动到预览视图中；
3. 点击预览按钮。
![](https://raw.githubusercontent.com/renshengqiang/renshengqiang.github.io/master/images/UnityAnimation/animationPreview.png)

##动画中插入事件

1. 选中骨骼动画文件；
2. Inspector 面板中选中 Animations 标签；
![](https://raw.githubusercontent.com/renshengqiang/renshengqiang.github.io/master/images/UnityAnimation/animationEvent.png)
3. 选中其中的一个 Animation Clip；
4. 往下拖动，找到 Events;
![](https://raw.githubusercontent.com/renshengqiang/renshengqiang.github.io/master/images/UnityAnimation/animationAddEvent.png)
5. 添加或者删除。

添加的事件会通过 GameObject.SendMessage 的方式发送到 GameObject 上面所有的 Component 上。

##Refs:
1. [三维图形引擎关键技术的研究和实现](http://d.wanfangdata.com.cn/Thesis/D611521)
2. [三维图形引擎关键技术的研究和实现-豆丁网](http://www.docin.com/p-1291631714.html)
3. [Unity Manual - Animator](http://docs.unity3d.com/460/Documentation/Manual/Animator.html)