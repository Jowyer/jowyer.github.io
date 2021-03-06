---
layout: post
title:  Scan QR Code
date:   2014-12-24 22:51:08
categories: iOS
---
iOS 7 给**AV Foundation**添加了不少新的功能，其中就包括今天要谈到的Barcode reading support。

## Introduction

### AV Foundation

{% highlight objc %}
@import AVFoundation;
{% endhighlight %}

The **AV Foundation** framework provides an Objective-C interface for managing and playing audio-visual media in iOS and OS X applications. 

It is a way to manage camera and microphone input, process digital media, and send output to the disk or the screen — a process collectively known as **media graph processing**.

### AVCaptureSession
You use an `AVCaptureSession` object to coordinate the flow of data from AV input devices to outputs. 

It is the core media handling class in AV Foundation. It talks to the hardware to retrieve, process, and output video. A capture session wires together inputs and outputs, and controls the format and resolution of the output frames.

### AVCaptureDevice
An `AVCaptureDevice` object represents a physical capture device and the properties associated with that device. 

It encapsulates the physical camera on a device. Modern iPhones have both front and rear cameras, while other devices may only have a single camera.

### AVCaptureDeviceInput
You use an `AVCaptureDeviceInput` to capture data from an `AVCaptureDevice` object.

To add an `AVCaptureDevice` to a session, wrap it in an `AVCaptureDeviceInput`. A capture session can have multiple inputs and multiple outputs.

### AVCaptureMetadataOutput
An `AVCaptureMetadataOutput` object intercepts metadata objects emitted by its associated capture connection and forwards them to a delegate object for processing. You can use instances of this class to process specific types of metadata included with the input data.

It provides a callback to the application when metadata is detected in a video frame. AV Foundation supports two types of metadata: *machine readable codes* and *face detection*.

### AVCaptureVideoPreviewLayer
`AVCaptureVideoPreviewLayer` is a subclass of CALayer that you use to display video as it is being captured by an input device.

It provides a mechanism for displaying the current frames flowing through a capture session; it allows you to display the camera output in your UI.

## Usage
All you need to do is set up an `AVCaptureMetaDataOutput` as the output of an `AVCaptureSession`, and implement the `captureOutput:didOutputMetadataObjects:fromConnection:` method accordingly:

{% highlight objc %}
@import AVFoundation;

AVCaptureSession *session = [[AVCaptureSession alloc] init];

AVCaptureDevice *device = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];

NSError *error = nil;
AVCaptureDeviceInput *input = [AVCaptureDeviceInput deviceInputWithDevice:device
                                                                    error:&error];
if (input && [session canAddInput:input]) {
    [session addInput:input];
} else {
    NSLog(@"Error: %@", error);
}

/* AV Foundation is designed for high throughput and low latency; 
therefore any processing and analysis tasks should be
moved off of the main thread if at all possible.
*/
dispatch_queue_t metadataQueue = dispatch_queue_create("com.jw.metadata", 0);

AVCaptureMetadataOutput *output = [[AVCaptureMetadataOutput alloc] init];
[output setMetadataObjectsDelegate:self queue:metadataQueue]; //AVCaptureMetadataOutputObjectsDelegate
if ([session canAddOutput:output]) {
    [session addOutput:output];
}
output.metadataObjectTypes = output.availableMetadataObjectTypes;

UIView *previewView = [[UIView alloc] initWithFrame:self.view.bounds];
[self.view addSubview:previewView];
    

AVCaptureVideoPreviewLayer *previewLayer = [[AVCaptureVideoPreviewLayer alloc] initWithSession:session];
previewLayer.videoGravity = AVLayerVideoGravityResizeAspectFill;
previewLayer.frame = previewView.bounds;
[previewView.layer addSublayer:previewLayer];

[session startRunning];
// Sessions should run only when the view controller is on screen.
// [session stopRunning];

#pragma mark - AVCaptureMetadataOutputObjectsDelegate
- (void)captureOutput:(AVCaptureOutput *)captureOutput
didOutputMetadataObjects:(NSArray *)metadataObjects
       fromConnection:(AVCaptureConnection *)connection
{
    NSString *codeString = nil;
    for (AVMetadataObject *metadata in metadataObjects) {
        if ([metadata isKindOfClass: [AVMetadataMachineReadableCodeObject class]]) {
            codeString = [(AVMetadataMachineReadableCodeObject *)metadata stringValue];
            NSLog(@"codeString : %@", codeString);
            break;
        }
    }
    
    dispatch_sync(dispatch_get_main_queue(), ^{
        // ...
    });
}
{% endhighlight %}

## Demo
做了一个类似微信扫码的demo，放在[GitHub](https://github.com/Jowyer/QRCodeDemo)上了。

## Reference
[RayWenderlich](http://www.raywenderlich.com/store/ios-7-by-tutorials)

[NSHipster](http://nshipster.com/ios7/)


