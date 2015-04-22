---
layout: post
title:  AutoLayout & UIScrollView
date:   2015-1-12 23:25:30
categories: iOS
---

## Update

今天看到了一个更直观更简单的[办法](http://spin.atomicobject.com/2014/03/05/uiscrollview-autolayout-ios/)：

1. 为`scrollView`添加一个`contentView`，设置好它们之间的constraints，`contentView`务必要和`scrollView`的四个边界都要有constraint。
2. 定义`contentView`的**Width Constraint**和**Height Constraint**，并将constraint的`Placeholder`的属性勾上：**Remove at build time**。此时仅仅用作布局，在build time时是被remove掉的。
3. 在**Class**的`-viewDidLayoutSubviews`方法中手动添加上一步的**Width Constraint**和**Height Constraint**。

{% highlight objc %}
[self.scrollContentView addConstraint:[NSLayoutConstraint constraintWithItem:self.scrollContentView attribute:NSLayoutAttributeWidth relatedBy:NSLayoutRelationEqual toItem:nil attribute:NSLayoutAttributeNotAnAttribute multiplier:1.0 constant:self.scrollView.frame.size.width * 2]];
[self.scrollContentView addConstraint:[NSLayoutConstraint constraintWithItem:self.scrollContentView attribute:NSLayoutAttributeHeight relatedBy:NSLayoutRelationEqual toItem:nil attribute:NSLayoutAttributeNotAnAttribute multiplier:1.0 constant:self.scrollView.frame.size.height]];
{% endhighlight %}

如果对于**UISplitViewController**，在`-presentDetailViewController`后，原有detailViewController的`frame`会有所改变，那么可以用 delay执行 的方式来解决下。

{% highlight objc %}
[self performSelector:@selector(addConstraints) withObject:nil afterDelay:0.1];
{% endhighlight %}


## Preface

在对**UIScrollView**进行**AutoLayout**布局时，会比使用其他控件更麻烦一些。

**UIScrollView**有一个`contentSize`属性，其定义了**UIScrollView**可滚动内容的大小。以前用纯代码的方式时，我们会直接对这个属性赋值，定义其大小。在**AutoLayout**中，我们不会直接使用这个属性，而`contentSize`是由其**内容**的约束来实现自动赋值。所以，我们需要对**UIScrollView**和它的**content**分别添加约束。

## Masonry

Apple[官方文档](https://developer.apple.com/library/ios/technotes/tn2154/_index.html)中采用了[Visual Format Language](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/VisualFormatLanguage/VisualFormatLanguage.html)的方式来定义和使用`NSLayoutConstraints`，不过，我们还可以有更好的使用方式：[Masonry](https://github.com/Masonry/Masonry)

> Harness the power of AutoLayout NSLayoutConstraints with a simplified, chainable and expressive syntax. 

## Usage
1. 在**Storyboard**中添加**UIScrollView**，并添加其constraints。
2. 新建**ContentView.xib**，继承自**UIView**，作为**UIScrollView**的内容视图，方便在**IB**中编辑。留意将`Simulated Metrics Size`设置为`Freeform`。
3. 在**ContentView.xib**中布局各个**subview**，并添加相应的constraints。
4. 新建一个Class：**ContentView**，并将**ContentView.xib**的类型定义为**ContentView**。
5. 将**ContentView.xib**中需要依据**UIScrollView**大小进行改变的constraints，在**ContentView.h**中建立**IBOutlet**连接。

{% highlight objc %}
@interface ContentView : UIView
@property (weak, nonatomic) IBOutlet NSLayoutConstraint *heightConstraint;
@property (weak, nonatomic) IBOutlet NSLayoutConstraint *widthConstrait;
@end
{% endhighlight %}

6 . 实例化**ContentView**，并建立constraint让`contentView`的四个边界与`scrollView`的四个边界相同。

{% highlight objc %}
- (void)viewDidLoad {
    [super viewDidLoad];
    
    contentView = [[[NSBundle mainBundle] loadNibNamed:@"ContentView" owner:self options:nil] objectAtIndex:0];
    [self.scrollView addSubview:contentView];
    
    [self.scrollView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.edges.equalTo(contentView);
    }];
}
{% endhighlight %}

7 . 在**AutoLayout**布局生效后，根据**scrollView**的实际宽高调整**contentView**的外连constraint，再借此调整**contentView**自身宽高的大小。

{% highlight objc %}
- (void)viewDidLayoutSubviews {
    contentView.widthConstrait.constant = self.scrollView.frame.size.width;
    contentView.heightConstraint.constant = self.scrollView.frame.size.height;
    
    [contentView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.width.equalTo([NSNumber numberWithFloat:contentView.widthConstrait.constant * 3]);
        make.height.equalTo([NSNumber numberWithFloat:contentView.heightConstraint.constant]);
    }];
    
    [super viewDidLayoutSubviews];
}
{% endhighlight %}

## Demo

做了一个demo，放在[GitHub](https://github.com/Jowyer/UIScrollViewAutoLayoutDemo)上了。

## Reference
[tuts+](http://code.tutsplus.com/tutorials/introduction-to-the-visual-format-language--cms-22715)

[十七蝉](http://blog.shiqichan.com/UIScrollView-And-Autolayout/)


