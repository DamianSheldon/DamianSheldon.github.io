---
layout: post
title: "(翻译)Variable-Sized Items in UICollectionView"
date: 2014-11-04 11:27:12 +0800
comments: true
categories: [Archives, iOS Development]
keywords: UICollectionView, Auto Layout, UICollectionViewCell
discription: How to layout Variable-Sized Items in UICollectionView
---

###基本训练  
我们以“Single View Application”为模板新建一个工程。在ViewController.xib上添加一个UICollectionView让它覆盖整个view。

<img name="Variable-Sized_Items_in_UICollectionView01" src="/images/Variable-Sized_Items_in_UICollectionView01.png" width="1110" height="783">  

Collection View和它的祖父类似，有delegate和dataSource两个outlet,我们把它们连接到“File’s Owner”,这里的“File’s Owner”是ViewController类。这样Collection View的内容和交互就受ViewController控制。我们也需要一个outlet来引用Collection View,所以用Assistant editor增加一个。刚在ViewController.h中添加的outlet property可以是weak，因为作为ViewController的视图的子视图，它也会被充分的保留。

###单元格

我们需要设计一个Cell原型来显示内容。如果你的Collection View是在View controller’s XIB ,你必须在先在代码中注册Cell才能使用。如果你是在Storyboard中新建的CollectionView，并在CollectionView的区域内创建的原型Cell,那么你就不需要注册Cell identifier了。因为我们并没有从Storyboard中开始，所以需要手动来创建。

我们在Interface Builder中创建一个原型Cell，设置它的背影为白色，添加一个UILabel，添加Label到父视图边缘的约束，分别是5 points的距离。因为我们想让Label的文字来决定它的尺寸，所以选中Label,然后从Editor menu中选择“Size to Fit Content”。后面我们会看到它是否会按照我们想像的那样工作。

<img name="Variable-Sized_Items_in_UICollectionView02" src="/images/Variable-Sized_Items_in_UICollectionView02.png" width="964" height="400">  

我们设置Identifier为“TagCell”以便我们能在设计中引用。

<!-- more -->

这里我们遇到了第一个问题。如果你没有使用Storyboards,并没有办法让Collection View在相同的NIB文件中使用Collection View Cell。有两种可行的方法注册Cell:
```objective-c
	– registerClass:forCellWithReuseIdentifier:
	– registerNib:forCellWithReuseIdentifier:
```

方法一为指定的Reuse identifier实例化某个特定的类，如果我们是通过代码来创建Cell的视图层级，我们应该使用这个方法；    
方法二需要一个NIB,而且NIB文件中只有Cell一个元素。在Storyboard中使用Collection Views避免了这个麻烦，这是它工作量更小的第二个重要原因。


为了解决这个问题，我们创建一个“empty Interface Builder Document”，命名为TagCollectionViewCell。感激零涕我们可以简单地CMD+X Cell设计从一个ViewController NIB中，然后CMD+V它到一个空的文档中。

<img name="Variable-Sized_Items_in_UICollectionView03" src="/images/Variable-Sized_Items_in_UICollectionView03.png" width="791" height="229">  

下一步我们将试着注册Cell在Collection View中使用，并想看到显示一定数量的元素。

###注册设计的单元格

任何时候我们想要一个标识符为“TagCell”的Cell都需要注册设计好的NIB文件，所以我们在ViewController.m文件的viewDidLoad中加入如下代码。
```objective-c
	UINib *cellNib = [UINib nibWithNibName:@"TagCollectionViewCell" bundle:nil];
	[self.collectionView registerNib:cellNib forCellWithReuseIdentifier:@"TagCell"];
```

我们仅需要实现的更多方法属于UICollectionViewDataSource协议，这些方法确保单元格正确显示。

```objective-c
#pragma mark - UICollectionView
 
- (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section
{
   return 100;
}
 
- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath
{
   UICollectionViewCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:@"TagCell" forIndexPath:indexPath];
 
   return cell;
}
```

现在启动应用，我们看到100个元素全部是使用我们大概还没获奖的设计原型。

<img name="Variable-Sized_Items_in_UICollectionView04" src="/images/Variable-Sized_Items_in_UICollectionView04.png" width="396" height="744">  

一眼就看出所有的元素都拥有相同的大小，准确的50x50 points。这是因为我们没有修改Interface Builder的默认值。

<img name="Variable-Sized_Items_in_UICollectionView05" src="/images/Variable-Sized_Items_in_UICollectionView05.png" width="261" height="273">  

但是在我们为每个元素指定大小之前，让我们首先来完善一下我们的设计。我们移除白色的背影，新建一个TagCollectionViewCell类，并且将它指定为元素的类。

###漂亮的单元

与其依赖iOS用单元的背影颜色填充整个矩形，我们想用一个圆角的矩形包围我们的标签。

```objective-c
- (void)drawRect:(CGRect)rect
{
   // inset by half line width to avoid cropping where line touches frame edges
   CGRect insetRect = CGRectInset(rect, 0.5, 0.5);
   UIBezierPath *path = [UIBezierPath bezierPathWithRoundedRect:insetRect cornerRadius:rect.size.height/2.0];
 
   // white background
   [[UIColor whiteColor] setFill];
   [path fill];
 
   // red outline
   [[UIColor redColor] setStroke];
   [path stroke];
}
```

注意：你需要设置单元的contentMode为redraw,否则修改单元的大小不会触发重绘这个漂亮的背影。

结果看起来像下面这样，有部分黑色的背影发光渗透。注意我们需要轻微缩进一点再绘画轮廓，因为Quartz将裁剪红色圆圈靠近视图边界的部分。

<img name="Variable-Sized_Items_in_UICollectionView06" src="/images/Variable-Sized_Items_in_UICollectionView06.png" width="394" height="110">  

下一步是让单元的大小适合现在标签的内容。

###逐个元素指定大小

当然我们并不满足用1）一个静态值指定所有的元素大小，所以2）想让大小随我们单元的内容自动调整。还记得我们指定元素标签依靠文本大小来决定自身大小吗？视图的边缘距离标签的边缘是固定的5 points。因此如果有一种方法能得到元素的实际大小，那将十分酷。更酷的是如果我们能告诉collection view用这些值来指定元素的大小。


LLDB的一个快速测试表明元素返回一个-1，-1的固定内容大小值，这个未定义是一样的。我们在TagCollectionViewCell的头文件中为Label加一个outlet,以便我们能找到UILabel返回它们显示当前文本需要的大小。

因为我们知道约束四周的空白，我们把它们和Label的固定内容大小相加就得到了元素的大小。
```objective-c
// cache for margins configured via constraints in XIB
static CGSize _extraMargins = {0,0};
 
@implementation TagCollectionViewCell
 
- (CGSize)intrinsicContentSize
{
   CGSize size = [self.label intrinsicContentSize];
 
   if (CGSizeEqualToSize(_extraMargins, CGSizeZero))
   {
      // quick and dirty: get extra margins from constraints
      for (NSLayoutConstraint *constraint in self.constraints)
      {
         if (constraint.firstAttribute == NSLayoutAttributeBottom || constraint.firstAttribute == NSLayoutAttributeTop)
         {
            // vertical spacer
            _extraMargins.height += [constraint constant];
         }
         else if (constraint.firstAttribute == NSLayoutAttributeLeading || constraint.firstAttribute == NSLayoutAttributeTrailing)
         {
            // horizontal spacer
            _extraMargins.width += [constraint constant];
         }
      }
   }
 
   // add to intrinsic content size of label
   size.width += _extraMargins.width;
   size.height += _extraMargins.height;
 
   return size;
}
 
@end
```

该方法从我们在Interface Builder设置的约束中得到实际的留白宽度。它允许我们在IB中调整大小而不需要在代码中改变常量或宏定义。因为我们不打算修改任何约束，只是在方法第一次被调用时懒散的设置一个静态_extraMargins作为常量。

现在困难的部分是从Collection view中出列一个元素并得到大小。它之所以困难是因为方法会调用数据源的collectionView:cellForItemAtIndexPath:，还会调用collectionView:layout:sizeForItemAtIndexPath:方法。这里禁止我们从后一种方法中出列一个元素，否则会导致无限循环。

有些人从模型对象中得到数据，然后cell有一个类方法来计算需要的大小。然而这没有利用我们想要在Interface Builder中设置的约束。对于这个先有鸡还是先有蛋的问题，我能想到的最简便的方法是使用一个单独的元素作为模板，然后使用它新鲜出炉的intrinsicContentSize方法。

```objective-c
@implementation ViewController
{
   TagCollectionViewCell *_sizingCell;
}
 
- (void)viewDidLoad
{
   [super viewDidLoad];
 
   UINib *cellNib = [UINib nibWithNibName:@"TagCollectionViewCell" bundle:nil];
   [self.collectionView registerNib:cellNib forCellWithReuseIdentifier:@"TagCell"];
 
   // get a cell as template for sizing
   _sizingCell = [[cellNib instantiateWithOwner:nil options:nil] objectAtIndex:0];
}
 
#pragma mark - UICollectionView
 
- (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section
{
   return 100;
}
 
- (void)_configureCell:(TagCollectionViewCell *)cell forIndexPath:(NSIndexPath *)indexPath
{
   if (indexPath.row%2)
   {
      cell.label.text = @"A";
   }
   else if (indexPath.row%3)
   {
      cell.label.text = @"longer";
   }
   else 
   {
      cell.label.text = @"much longer";
   }
}
 
- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath
{
   TagCollectionViewCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:@"TagCell" forIndexPath:indexPath];
 
   [self _configureCell:cell forIndexPath:indexPath];
 
   return cell;
}
 
- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout sizeForItemAtIndexPath:(NSIndexPath *)indexPath
{
   [self _configureCell:_sizingCell forIndexPath:indexPath];
 
   return [_sizingCell intrinsicContentSize];
}
 
@end
```

为Collection view注册了NIB之后，我创建了个元素实例并赋值给了我的实例变量 _sizingCell。为了得到实际元素单元，我们有一个 _configureCell:forIndexPath:方法，它作用于出列可复用的实例上。为了计算大小，我们应用这个相同的方法到我们的sizing cell,以便我们得到正确的intrinsicContentSize。

###Once More With Auto Layout

当我救助我的推友们关于如何得到基于约束的元素的大小，[Martin Pilkington](https://twitter.com/pilky)迅速将我指向-systemLayoutSizeFittingSize:。一开始在应用中失败了，得到的全是{0,0}。但是当我写完上述解决办法，沉下心来又试了一次。

这一次它正常工作了。所需要做的是替换下面方法：
```objective-c
- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout sizeForItemAtIndexPath:(NSIndexPath *)indexPath
{
   [self _configureCell:_sizingCell forIndexPath:indexPath];
 
   return [_sizingCell systemLayoutSizeFittingSize:UILayoutFittingCompressedSize];
}
```

这个方法决定一个最接近传入大小的布局大小。两个标准值都可行，UILayoutFittingCompressedSize得到基于布局的最小大小，UILayoutFittingExpandedSize得到最大大小。

有了它我们可以很愉快地利用我们之前在intrinsicContentSize做的工作。我们也可以完全使用布局约束来进一步限制元素各个部分的大小。例如说：你可能想要短标签不要窄于某个宽度；如果某个标签太长了就让它截断。

<img name="Variable-Sized_Items_in_UICollectionView07" src="/images/Variable-Sized_Items_in_UICollectionView07.png" width="394" height="214">  

结果如下，证明确实是我们想要的。

<img name="Variable-Sized_Items_in_UICollectionView08" src="/images/Variable-Sized_Items_in_UICollectionView08.png" width="396" height="744">  

是不是很酷？让我们再强调一次：你不需要在Cell方法中计算大小，本质是重复了auto layout将要执行的计算。你只需要向系统要。

###总结

在这篇博文中，我向你展示了如何为Collection view创建一个来自NIB的Cell。备选方法是在Storyboard中使用Collection view，它在某些方面更方便。这些Cell没有放在单独的XIB文件中，但是是在Collection view的结构层级中。探索如何得到一个sizing cell就作为练习留给读者了。


我们发现UILabel暴露一个intrinsicContentSize方法十分方便，如果你在Interface Builder指定了auto layout，它会用来计算大小。我们探索了一种方法，它使用一个cell实例作为模板来决定最佳的元素大小。然后我们更进一步运用了一个方法告诉我们基于布局约束和固定内容大小的完美尺寸。

例子的代码放在GitHub的[Cocoanetics Examples](https://github.com/Cocoanetics/Examples) 仓库中。

###原文
[Variable-Sized Items in UICollectionView](http://www.cocoanetics.com/2013/08/variable-sized-items-in-uicollectionview/)

