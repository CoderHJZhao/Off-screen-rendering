### 圆角绘制的几种方法

---
- 设置Layer的cornerRadius属性

如：

```
imageview.cornerRadius = 5.0f;
imageview.layer.masksToBounds = Yes;
```
但这种方法会导致视图的离屏渲染，极大的降低了视图滑动的流畅性，那什么是离屏渲染呢？又该怎么解决呢？


> Off-Screen Rendering：
离屏渲染，指的是GPU在当前屏幕缓冲区以外新开辟一个缓冲区进行渲染操作。由上面的一个结论视图和圆角的大小对帧率并没有什么卵影响，数量才是伤害的核心输出啊。可以知道离屏渲染耗时是发生在离屏这个动作上面，而不是渲染。为什么离屏这么耗时？原因主要有创建缓冲区和上下文切换。创建新的缓冲区代价都不算大，付出最大代价的是上下文切换。

除了绘制圆角的这种方式会造成离屏渲染外，以下几种方式也会造成离屏渲染，如：

- shouldRasterize（光栅化）
- masks（遮罩）
- shadows（阴影）
- edge antialiasing（抗锯齿）
- group opacity（不透明）
- 复杂形状设置圆角等
- 渐变
- Text（UILabel, CATextLayer, Core Text, etc）...
以上几种情况的详细介绍，大家可以参阅：[iOS 离屏渲染，的研究](http://www.jianshu.com/p/6d24a4c29e18)

那么为了避免绘制圆角造成的离屏渲染，我们可以用以下几种方式绘制远角：

- 通过shapeLayer设置

通过bezizerpath设置一个路径，加到目标视图的layer上。代码如下：

```

 // 创建一个view
    UIView showView = [[UIView alloc] initWithFrame:CGRectMake(100, 100, 100, 100)];
    [self.view addSubview:showView];     
    showView.backgroundColor = [UIColor whiteColor];
    showView.alpha = 0.5;     // 贝塞尔曲线(创建一个圆)
     UIBezierPath path = [UIBezierPath     bezierPathWithArcCenter:CGPointMake(100 / 2.f, 100 / 2.f)
                                                        radius:100 / 2.f                                                    startAngle:0 
                                                      endAngle:M_PI  2
                                                     clockwise:YES];

     CAShapeLayer *layer = [CAShapeLayer layer];
     layer.frame = showView.bounds;
     layer.path = path.CGPath;
     [showView.layer addSublayer:layer];

```
- 通过BezierPath设置

BezierPath的实现方式继承UIView，自己实现一个customview,代码如下。
```
- (instancetype)initWithFrame:(CGRect)frame {

    if (self = [super initWithFrame:frame]) {

    }    return self;
}
- (void)drawRect:(CGRect)rect { 
   // Drawing code 
    CGRect bounds = self.bounds;
    [[UIColor whiteColor] set];
    UIRectFill(bounds);

    [[UIBezierPath bezierPathWithRoundedRect:rect cornerRadius:CGRectGetWidth(bounds)/2.0] addClip];
    [self.image drawInRect:bounds];
}
```
Tips：在声明drawRect的时候会耗用系统极大的内存和资源，所以在一般的情况下，不太建议声明这个方法。

- 通过贴图的方式设置

贴图的方式是使用一个中间是圆形镂空的图覆盖在需要圆角化的图片的上方。代码如下：（求听话的美工一枚）
```
- (instancetype)initWithStyle:(UITableViewCellStyle)style reuseIdentifier:(NSString )reuseIdentifier {

    if (self = [super initWithStyle:style reuseIdentifier:reuseIdentifier]) {

        [self.contentView addSubview:self.avatarImage];
        [self.contentView addSubview:self.maskImage];
    }
    return self;   }

- (UIImageView *)avatarImage {

    if (!_avatarImage) { 

        _avatarImage = [[UIImageView alloc] initWithFrame:CGRectMake(20,10, avatarDiameter, avatarDiameter)];
        _avatarImage.backgroundColor = [UIColor grayColor];
        _avatarImage.contentMode = UIViewContentModeScaleAspectFit;
        [_avatarImage setImage:[UIImage imageNamed:@"test.jpg"]];
    }
    return _avatarImage;}//中心镂空的图
- (UIImageView *)maskImage {  

    if (!_maskImage) { 

        _maskImage = [[UIImageView alloc] initWithFrame:CGRectMake(20,10, avatarDiameter, avatarDiameter)]; 
        _maskImage.contentMode = UIViewContentModeScaleAspectFit; 
        [_maskImage setImage:[UIImage imageNamed:@"corner_circle.png"]];    }
    return _maskImage;} 
```
以上就是几种常见的避免离屏渲染的绘制圆角的方法，如有补充，欢迎交流！
