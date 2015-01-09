---
layout: post
title:  Custom Font
date:   2015-1-9 12:55:09
categories: iOS
---
简单记录下在项目中使用自定义字体的方法。

## Import

1. 将字体文件直接拖入Xcode。可以建立独立文件夹存放：`Resources/Fonts/`
2. 在**info.plist**中新建字段：`Fonts provided by application`
3. 在**item**所对应的**Value**值中填入字体文件的名称如：*TitilliumText25L.otf*

## Usage
1. **Storyboard**中可直接使用。
2. **Launch Screen**中用还有[问题](http://stackoverflow.com/questions/25794314/using-custom-fonts-with-xcode-6-ios-8-interface-builder-launch-screen)，在Xcode中看见已经是自定义的字体，但是运行起来之后，却用的是**System Font**，应该是Xcode的一个bug。
3. 代码中使用，注意是用**font name**而不是**font family name**：

{% highlight objc %}
myLabel.font = [UIFont fontWithName:@"TitilliumText25L-999wt" size:14];
{% endhighlight %}

## Available Fonts

{% highlight objc %}
for (NSString* family in [UIFont familyNames])
{
    NSLog(@"%@", family);
        
    for (NSString* name in [UIFont fontNamesForFamilyName: family])
  {
	NSLog(@"  %@", name);
  }
}
{% endhighlight %}



## Reference
[CODEwithChris](http://codewithchris.com/common-mistakes-with-adding-custom-fonts-to-your-ios-app/)


