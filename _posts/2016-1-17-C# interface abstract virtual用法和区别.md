---
layout: post
author: tnqiang
titile: C# interface virtual 和 abstract的用法和区别
category: C#
tag: C#
---
##interface用法

{% highlight C# %}
public interface IXXX
{
	void DoXXX();
}

public YYY : interface
{
	public void DoXXX
	{
		...
	}
}

{% endhighlight %}

##virtual用法

{% highlight C# %}
public XXX
{
	public virtual void DoXXX()
	{
		...
	}
}

public YYY : XXX
{
	public override void DoXXX()
	{
		...
		base.DoXXX();
	}
}

{% endhighlight %}

##abtract 用法

{% highlight C# %}
public abstract class XXX
{
	public void Init()
	{
		...
		DoXXX();
		...
	}
	public abstract void DoXXX();
}

public class YYY
{
	public override void DoXXX()
	{
		...
	}
}

{% endhighlight %}

##interface 与 abstract 的异同

###同
- 都能修饰类名
- interface 和 abstract 类都不能实例化

###异
- abstract可以修改函数，interface不能
- interface类的接口不需要声明权限，实现必须是 public 的；abstract 函数的实现可以是 protected 或者 public
- interface类只能声明函数，abstract 类可以使用 abstract 声明或者直接来实现一个方法，并且可以有变量

###abstract 和 virtual 的异同

###同
- 子类实现都使用 override 关键字进行重写
- 使用 override 关键字的函数最后调用的都是子类的方法

###异
- abstract 可以声明类，virual 不能
- abstract 只能进行函数声明，virtual 父类必须要有实现
- virtual函数对子类没有什么语法要求，abstract 必须要有实现体

###一般用法
1. 只想声明方法，不想任何实现，使用 interface 即可
2. 既想实现方法，又想定义接口，使用 abstract 类 和 abstract 方法
3. 既想实现方法，又想让子类来改变某些行为，使用 virtual 方法