---
layout: post
title:  UIAlertController
date:   2014-10-30 21:47:31
categories: iOS
---
决定以后的博客尽量用Swift来写，还是要尽快转Swift啊。

`UIAlertController`也是iOS 8推出的一个新类，用来替代 `UIActionSheet` and `UIAlertView`。这样统一到一个类里，使用也更方便了。

> A UIAlertController object displays an alert message to the user. This class replaces the UIActionSheet and UIAlertView classes for displaying alerts.

## init(title:message:preferredStyle:)
设置好alertController的标题，正文，及出现的形式（Alert，ActionSheet）后，通过此方法进行初始化。

{% highlight swift %}
let title = "Breaking News"
let message = "Oops! iPhone 6 is sold out everywhere."
let alertController = UIAlertController(title: title, message: message, preferredStyle: .Alert)
{% endhighlight %}

## Presentation
完成初始化后，通过**presentViewController(_:animated:completion:)**方法将alert展示出来。

{% highlight swift %}
presentViewController(alertController, animated: true, nil)
{% endhighlight %}

## UIAlertAction
这样显示出来的alert是没有按钮的，要加上按钮及处理按钮按下后的逻辑，iOS 8采用了一种全新的方式。

要处理`UIAlertView`和`UIActionSheet`的按下逻辑，我们通常需要实现`<UIAlertViewDelegate>`和`<UIActionSheetDelegate>`（deprecated in iOS 8）。不过从今往后，我们只需要和`UIAlertAction`打交道就可以了。

> A UIAlertAction object represents an action that can be taken when tapping a button in an alert. You use this class to configure information about a single action, including the title to display in the button, any styling information, and a handler to execute when the user taps the button. 

### init(title:style:handler:)
设置好alertAction按钮的名称，样式，和按下后的操作block，通过此方法初始化`UIAlertAction`。

{% highlight swift %}
let cancelActioin = UIAlertAction(title: "OK, I know", style: .Cancel) { action in
	NSLog("Cancel Button Tapped")
}
{% endhighlight %}

## addAction(_:)
将`UIAlertAction`附加到alert或者action sheet上。

当有多个actions的时候，添加的顺序确定了显示的顺序。
{% highlight swift %}
let preorderActioin = UIAlertAction(title: "Preorder it anyway!", style: .Destructive) { action in
	NSLog("Preorder Button Tapped")
}
let nokiaActioin = UIAlertAction(title: "I will buy a Nokia", style: .Default) { action in
	NSLog("Nokia Button Tapped")
}
        
alertController.addAction(cancelActioin)
alertController.addAction(preorderActioin)
alertController.addAction(nokiaActioin)
{% endhighlight %}

![Alert](/images/UIAlertController_1.png)

`.Destructive`和`.Default`的button，越先添加的就越近正文；`.Cancel`的button，离正文最远。

如果只有2个button，文本较少而能够并排排列的话，`.Cancel`则和其余类型的一样，也遵循越先添加的就越近正文，位于**并排的左侧**。

`.Destructive`的button，文字是**红色**。

`.Cancel`的button，只能有**一个**。

## addTextFieldWithConfigurationHandler(_:)
在alert里面添加一个text field。可以在其后的block里完成对text field的设置。

如要添加多个text field，只需多次调用此方法。

此方法只适用于alert，不能用于action sheet。
{% highlight swift %}
alertController.addTextFieldWithConfigurationHandler { textField in
	textField.textColor = UIColor.blueColor()
}
alertController.addTextFieldWithConfigurationHandler { textField in
	textField.placeholder = "fill me"
}
{% endhighlight %}

## About ActionSheet
通常将action sheet的`title`和`message`都设为`nil`。
{% highlight swift %}
let alertController = UIAlertController(title: nil, message: nil, preferredStyle: .ActionSheet)
{% endhighlight %}

