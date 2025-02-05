---
layout: post
title: "(翻译)Variable-Sized Items in UICollectionView"
date: 2014-11-04 11:27:12 +0800
comments: true
categories: [Archives, iOS Development]
keywords: UICollectionView, Auto Layout, UICollectionViewCell
description: How to layout Variable-Sized Items in UICollectionView
---

###基本训练  
我们以 “Single View Application” 为模板新建一个工程。在 ViewController.xib 上添加一个UICollectionView 让它覆盖整个 view。

<img name="Variable-Sized_Items_in_UICollectionView01" src="/images/Variable-Sized_Items_in_UICollectionView01.png" width="1110" height="783">  

Collection View 和它的祖父类似，有 delegate 和 dataSource 两个 outlet,我们把它们连接到 “File’s Owner”,这里的 “File’s Owner” 是 ViewController 类。这样 Collection View 的内容和交互就受 ViewController 控制。我们也需要一个 outlet 来引用 Collection View,所以用 Assistant editor 增加一个。刚在 ViewController.h 中添加的 outlet property 可以是weak，因为作为 ViewController 的视图的子视图，它也会被充分的保留。

###单元格

我们需要设计一个 Cell 原型来显示内容。如果你的 Collection View 是在 View controller’s XIB ,你必须在先在代码中注册 Cell 才能使用。如果你是在 Storyboard 中新建的 CollectionView，并在 CollectionView 的区域内创建的原型 Cell,那么你就不需要注册 Cell identifier 了。因为我们并没有从 Storyboard 中开始，所以需要手动来创建。

我们在 Interface Builder 中创建一个原型 Cell，设置它的背影为白色，添加一个 UILabel，添加 Label 到父视图边缘的约束，分别是 5 points 的距离。因为我们想让 Label 的文字来决定它的尺寸，所以选中 Label ,然后从 Editor menu 中选择 “Size to Fit Content”。后面我们会看到它是否会按照我们想像的那样工作。

<img name="Variable-Sized_Items_in_UICollectionView02" src="/images/Variable-Sized_Items_in_UICollectionView02.png" width="964" height="400">  

我们设置 Identifier 为 “TagCell” 以便我们能在设计中引用。

<!-- more -->

这里我们遇到了第一个问题。如果你没有使用 Storyboards,并没有办法让 Collection View 在相同的 NIB 文件中使用 Collection View Cell。有两种可行的方法注册 Cell:
```objective-c
	– registerClass:forCellWithReuseIdentifier:
	– registerNib:forCellWithReuseIdentifier:
```

方法一为指定的 Reuse identifier 实例化某个特定的类，如果我们是通过代码来创建 Cell 的视图层级，我们应该使用这个方法；    
方法二需要一个 NIB,而且 NIB 文件中只有 Cell 一个元素。在 Storyboard 中使用 Collection Views 避免了这个麻烦，这是它工作量更小的第二个重要原因。


为了解决这个问题，我们创建一个 “empty Interface Builder Document”，命名为 TagCollectionViewCell。感激零涕我们可以简单地 CMD+X Cell 设计从一个 ViewController NIB 中，然后 CMD+V 它到一个空的文档中。

<img name="Variable-Sized_Items_in_UICollectionView03" src="/images/Variable-Sized_Items_in_UICollectionView03.png" width="791" height="229">  

下一步我们将试着注册 Cell 在 Collection View 中使用，并想看到显示一定数量的元素。

###注册设计的单元格

任何时候我们想要一个标识符为 “TagCell” 的 Cell 都需要注册设计好的 NIB 文件，所以我们在 ViewController.m 文件的 viewDidLoad 中加入如下代码。
```objective-c
	UINib *cellNib = [UINib nibWithNibName:@"TagCollectionViewCell" bundle:nil];
	[self.collectionView registerNib:cellNib forCellWithReuseIdentifier:@"TagCell"];
```

我们仅需要实现的更多方法属于 UICollectionViewDataSource 协议，这些方法确保单元格正确显示。

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

一眼就看出所有的元素都拥有相同的大小，准确的 50x50 points。这是因为我们没有修改 Interface Builder 的默认值。

<img name="Variable-Sized_Items_in_UICollectionView05" src="/images/Variable-Sized_Items_in_UICollectionView05.png" width="261" height="273">  

但是在我们为每个元素指定大小之前，让我们首先来完善一下我们的设计。我们移除白色的背影，新建一个 TagCollectionViewCell 类，并且将它指定为元素的类。

###漂亮的单元

与其依赖 iOS 用单元的背影颜色填充整个矩形，我们想用一个圆角的矩形包围我们的标签。

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

注意：你需要设置单元的 contentMode 为 redraw,否则修改单元的大小不会触发重绘这个漂亮的背影。

结果看起来像下面这样，有部分黑色的背影发光渗透。注意我们需要轻微缩进一点再绘画轮廓，因为Quartz将裁剪红色圆圈靠近视图边界的部分。

<img name="Variable-Sized_Items_in_UICollectionView06" src="/images/Variable-Sized_Items_in_UICollectionView06.png" width="394" height="110">  

下一步是让单元的大小适合现在标签的内容。

###逐个元素指定大小

当然我们并不满足用1）一个静态值指定所有的元素大小，所以2）想让大小随我们单元的内容自动调整。还记得我们指定元素标签依靠文本大小来决定自身大小吗？视图的边缘距离标签的边缘是固定的 5 points。因此如果有一种方法能得到元素的实际大小，那将十分酷。更酷的是如果我们能告诉 collection view 用这些值来指定元素的大小。


LLDB 的一个快速测试表明元素返回一个 -1，-1 的固定内容大小值，这个未定义是一样的。我们在 TagCollectionViewCell 的头文件中为 Label 加一个 outlet,以便我们能找到 UILabel 返回它们显示当前文本需要的大小。

因为我们知道约束四周的空白，我们把它们和 Label 的固定内容大小相加就得到了元素的大小。
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

该方法从我们在 Interface Builder 设置的约束中得到实际的留白宽度。它允许我们在 IB 中调整大小而不需要在代码中改变常量或宏定义。因为我们不打算修改任何约束，只是在方法第一次被调用时懒散的设置一个静态 _extraMargins 作为常量。

现在困难的部分是从 Collection view 中出列一个元素并得到大小。它之所以困难是因为方法会调用数据源的 `collectionView:cellForItemAtIndexPath:`，还会调用 `collectionView:layout:sizeForItemAtIndexPath:` 方法。这里禁止我们从后一种方法中出列一个元素，否则会导致无限循环。

有些人从模型对象中得到数据，然后 cell 有一个类方法来计算需要的大小。然而这没有利用我们想要在 Interface Builder 中设置的约束。对于这个先有鸡还是先有蛋的问题，我能想到的最简便的方法是使用一个单独的元素作为模板，然后使用它新鲜出炉的 intrinsicContentSize 方法。

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

为 Collection view 注册了 NIB 之后，我创建了个元素实例并赋值给了我的实例变量 _sizingCell。为了得到实际元素单元，我们有一个 `_configureCell:forIndexPath:` 方法，它作用于出列可复用的实例上。为了计算大小，我们应用这个相同的方法到我们的 sizing cell,以便我们得到正确的 intrinsicContentSize。

###Once More With Auto Layout

当我求助我的推友们关于如何得到基于约束的元素的大小，[Martin Pilkington](https://twitter.com/pilky) 迅速将我指向 `-systemLayoutSizeFittingSize:`。一开始在应用中失败了，得到的全是{0,0}。但是当我写完上述解决办法，沉下心来又试了一次。

这一次它正常工作了。所需要做的是替换下面方法：
```objective-c
- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout*)collectionViewLayout sizeForItemAtIndexPath:(NSIndexPath *)indexPath
{
   [self _configureCell:_sizingCell forIndexPath:indexPath];
 
   return [_sizingCell systemLayoutSizeFittingSize:UILayoutFittingCompressedSize];
}
```

这个方法决定一个最接近传入大小的布局大小。两个标准值都可行，UILayoutFittingCompressedSize 得到基于布局的最小大小，UILayoutFittingExpandedSize 得到最大大小。

有了它我们可以很愉快地利用我们之前在 intrinsicContentSize 做的工作。我们也可以完全使用布局约束来进一步限制元素各个部分的大小。例如说：你可能想要短标签不要窄于某个宽度；如果某个标签太长了就让它截断。

<img name="Variable-Sized_Items_in_UICollectionView07" src="/images/Variable-Sized_Items_in_UICollectionView07.png" width="394" height="214">  

结果如下，证明确实是我们想要的。

<img name="Variable-Sized_Items_in_UICollectionView08" src="/images/Variable-Sized_Items_in_UICollectionView08.png" width="396" height="744">  

是不是很酷？让我们再强调一次：你不需要在 Cell 方法中计算大小，本质是重复了 auto layout 将要执行的计算。你只需要向系统要。

###总结

在这篇博文中，我向你展示了如何为 Collection view 创建一个来自 NIB 的 Cell。备选方法是在 Storyboard 中使用 Collection view，它在某些方面更方便。这些 Cell 没有放在单独的 XIB 文件中，但是是在 Collection view 的结构层级中。探索如何得到一个 sizing cell 就作为练习留给读者了。


我们发现 UILabel 暴露一个 intrinsicContentSize 方法十分方便，如果你在 Interface Builder 指定了 auto layout，它会用来计算大小。我们探索了一种方法，它使用一个 cell 实例作为模板来决定最佳的元素大小。然后我们更进一步运用了一个方法告诉我们基于布局约束和固定内容大小的完美尺寸。

例子的代码放在 GitHub 的[Cocoanetics Examples](https://github.com/Cocoanetics/Examples) 仓库中。

###原文
[Variable-Sized Items in UICollectionView](http://www.cocoanetics.com/2013/08/variable-sized-items-in-uicollectionview/)

