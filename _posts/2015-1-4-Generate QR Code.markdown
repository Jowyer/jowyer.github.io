---
layout: post
title:  Generate QR Code
date:   2015-1-4 22:47:55
categories: iOS
---
在前面的[博客](http://jowyer.github.io/ios/2014/12/25/Scan%20QR%20Code.html)中介绍了如何实现二维码扫描，今天继续说说怎么实现二维码的生成。

同样是在iOS 7以后，Apple给出了官方的解决方案 --- **CoreImage**中的[CIQRCodeGenerator](https://developer.apple.com/library/ios/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/filter/ci/CIQRCodeGenerator)。

## CIQRCodeGenerator
Generates a Quick Response code (two-dimensional barcode) from input data.

### Parameters
* ***inputMessage***:

  The data to be encoded as a QR code. An `NSData` object whose display name is *Message*.
* ***inputCorrectionLevel***

  A single letter specifying the error correction format. An `NSString` object whose display name is *CorrectionLevel*. Default value: **M**
  
## Usage

{% highlight objc %}

@import CoreImage;

- (UIImage *)qrImageForString:(NSString *)qrString
{
    // Need to convert the string to a UTF-8 encoded NSData object
    NSData *stringData = [qrString dataUsingEncoding:NSUTF8StringEncoding];
    
    // Create the filter
    CIFilter *qrFilter = [CIFilter filterWithName:@"CIQRCodeGenerator"];
    // Set the message content and error-correction level
    [qrFilter setValue:stringData forKey:@"inputMessage"];
    [qrFilter setValue:@"H" forKey:@"inputCorrectionLevel"];
    
    // Send the image back
    CIImage *outputImage = qrFilter.outputImage;
    UIImage *image = [UIImage imageWithCIImage:outputImage
                                         scale:1.
                                   orientation:UIImageOrientationUp];
    return image;
}
{% endhighlight %}

如果直接使用这个`UIImage`，会发现得到的二维码很模糊，这是因为通过filter的`outputImage`属性获得的`CIImage`，只满足了1pt resolution for the smallest squares。

我们可以通过`kCGInterpolationNone`的方式进行拉伸，来获得更清晰的图片。

{% highlight objc %}
- (UIImage *)resizeImage:(UIImage *)image
             withQuality:(CGInterpolationQuality)quality
                    rate:(CGFloat)rate
{
    UIImage *resized = nil;
    CGFloat width = image.size.width * rate;
    CGFloat height = image.size.height * rate;
    
    UIGraphicsBeginImageContext(CGSizeMake(width, height));
    CGContextRef context = UIGraphicsGetCurrentContext();
    CGContextSetInterpolationQuality(context, quality);
    [image drawInRect:CGRectMake(0, 0, width, height)];
    resized = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    
    return resized;
}
{% endhighlight %}

完整的流程：

{% highlight objc %}
NSString *string = @"http://jowyer.github.io/";

UIImage *qrImage = [self qrImageForString:string];

UIImage *resizedImage = [self resizeImage:qrImage
                              withQuality:kCGInterpolationNone
                                     rate:5.0];

self.myImageView.image = resizedImage;
{% endhighlight %}

## Reference
[ShinobiControls](http://www.shinobicontrols.com/blog/posts/2013/10/10/ios7-day-by-day-day-15-coreimage-filters)

[shu223](https://github.com/shu223/iOS7-Sampler)

[郭宇翔](http://blog.yourtion.com/custom-cifilter-qrcode-generator.html)

