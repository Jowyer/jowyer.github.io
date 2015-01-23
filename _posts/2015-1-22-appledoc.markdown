---
layout: post
title:  appledoc
date:   2015-1-22 20:57:28
categories: iOS
---
[appledoc](https://github.com/tomaz/appledoc)是一个基于Objective-C的文档生成器，用它可以很方便的写出类似Apple官方风格的文档。

> appledoc is command line tool that helps Objective-C developers generate Apple-like source code documentation from specially formatted source code comments. 


## Installation
从GitHub上把工程clone下来，再通过shell命令安装。

{% highlight bash %}
git clone git://github.com/tomaz/appledoc.git
cd appledoc
sudo sh install-appledoc.sh
{% endhighlight %}

## Documenting

Objective-C documentation is designated by a /** */ comment block (note the extra initial star), which **precedes** any `@interface` or `@protocol`, as well as any `method` or `@property` declarations.

以下面这段代码来说明：

{% highlight objc %}
/** Returns the absolute path of the Homebrew executable.
 *
 * @return The Homebrew executable path.
 */
- (NSString *)brewPath;

/** Sets the absolute path of the Homebrew executable.
 *
 * @param path The absolute path of the Homebrew executable. If `nil` the
 * default path `/usr/local/bin/brew` will be used.
 */
- (void)setBrewPath:(NSString *)path;

/**-----------------------------------------------------------------------------
 * @name Performing an Operation
 * -----------------------------------------------------------------------------
 */

/** Performs an operation.
 *
 * Operations are placed in a queue for execution and will always execute on
 * separate threads. Use setConcurrentOperations: to control how queued
 * operations are executed (i.e. concurrently, or serially).
 *
 * @param operation The operation to perform.
 * @param delegate The delegate object for the operation. The delegate will
 * receive delegate messages during execution of the operation when output is
 * generated and upon completion or failure of the operation.
 */
- (void)performOperation:(MRBrewOperation *)operation delegate:(id<MRBrewDelegate>)delegate;
{% endhighlight %}

### description

在**后面写`description`，更详细的`discussion`另起一行后再写。

{% highlight objc %}
/** description
 *
 * discussion
 *
 */
{% endhighlight %}

### @name

相当于`#pragma mark`的功能，给代码分段。

{% highlight objc %}
/**-----------------------------------------------------------------------------
 * @name Part 1
 * -----------------------------------------------------------------------------
 */
{% endhighlight %}

### @param

**@param [param] [Description]**: 对参数的描述。

{% highlight objc %}
* @param url The URL to access.
{% endhighlight %}

### @return
**@return [Description]**: 对返回值的描述。

{% highlight objc %}
* @return The return value of the method.
{% endhighlight %}

### @see
**@see [selector]**: 提供**see also**的指引，引出相关的内容。

{% highlight objc %}
* @see +manager
{% endhighlight %}

### @warning
**@warning [description]**: 提供警示内容。

{% highlight objc %}
* @warning Call out exceptional or potentially dangerous behavior
{% endhighlight %}

这些是一些常用的标识，更多的Javadoc-style @ labels看[这里](https://github.com/tomaz/appledoc/wiki/appledoc-docs-comments)。

## VVDocumenter-Xcode

[onevcat](http://www.onevcat.com/)开源的[VVDocumenter-Xcode](https://github.com/onevcat/VVDocumenter-Xcode)插件，安装之后重启Xcode，再输入 **///** 就能自动生成一个文档注释的模板，很方便。

## Usage
按照规范写好后，按住**option**键点击方法名，就能看到类似Apple文档的说明了。

如果想生成**.docset**文件，便于导入[Dash](http://kapeli.com/dash)之类的软件，只需要输入如下命令即可：

{% highlight  bash %}
appledoc -p testappledoc -c Jo2Studio --company-id com.jo2studio -o ./doc .
{% endhighlight %}

会将当前文件夹下的所有源文件进行扫描，生成文档，并在当前目录下的doc/文件夹下，新建一个文本文件，记录生成情况。
生成的文档文件默认会存放到：

**~/Library/Developer/Shared/Documentation/DocSets**

更多可用的参数可以用`appledoc --help`查看。或者查看官方的[wiki](https://github.com/tomaz/appledoc/wiki/appledoc-docs-examples-basic)。

p.s. 生成**.docset**文件后，在Xcode使用时，显示的内容反而不如之前多了。建议生成文档文件后将其移出默认系统文件夹。

## Reference
[NSHipster](http://nshipster.com/documentation/)

[唐巧](http://blog.devtang.com/blog/2012/02/01/use-appledoc-to-generate-xcode-doc/)


