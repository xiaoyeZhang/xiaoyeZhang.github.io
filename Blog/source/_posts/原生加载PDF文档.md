---
title: iOS 原生加载PDF文档(主要功能:跳转指定的页码)
date: 2023-11-24 16:43:50
---
## 前言

在实际的开发过程中，我们会遇到一些需要显示PDF的场景，比如官方文件（为了保证原有的格式显示正常，通常会做成PDF来展示），同时也会要求跳转到指定的PDF的页码中,这里我们来讨论一个展示PDF并且调整指定的页码的方式。

简单的介绍下加载PDF的几种方式(PDF可能是网络加载的，也可能是本地的)

- WKWebView加载本地或者网络pdf文档
- QLPreviewController加载pdf文档
- 用CGContext画pdf文档，并结合UIPageViewController展示
- 使用第三方框架vfr/Reader加载pdf文档

在这里我使用CGContext加上UICollectionView来实现加载PDF和跳转指定页码,(其他方法暂时不提及)

#### 第一步  创建UICollectionView
```
    self.TopHeight = self.navigationController.navigationBar.frame.size.height + [UIApplication sharedApplication].statusBarFrame.size.height;

    self.layout = [[UICollectionViewFlowLayout alloc] init];
        
    self.layout.itemSize = CGSizeMake(self.view.bounds.size.width, self.view.frame.size.height-self.TopHeight);

    [self.layout setScrollDirection:self.scrollDirection];//设置滑动方向为水平方向，也可以设置为竖直方向
        
    self.layout.minimumLineSpacing = 0;//设置item之间最下行距
        
    self.layout.minimumInteritemSpacing = 0;//设置item之间最小间距
        
    _CollectionView = [[UICollectionView alloc] initWithFrame:self.view.bounds collectionViewLayout:self.layout];
        
    _CollectionView.backgroundColor = [UIColor whiteColor];
    _CollectionView.pagingEnabled = self.PDFPagingEnabled;//设置集合视图一页一页的翻动
    _CollectionView.delegate = self;
    _CollectionView.dataSource = self;


    [self.view addSubview:self.CollectionView];//将集合视图添加到当前视图上

    [self.CollectionView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.top.mas_equalTo(self.TopHeight);
        make.left.right.equalTo(self.view);
        make.height.mas_equalTo(self.view.frame.size.height-self.TopHeight);
    }];

```

#### 第二步  创建UICollectionViewCell

在自定义的cell中对showView重写set方法,重写cell视图
```
    for(UIView *tempView in _contentScrollView.subviews) {
        
        [tempView removeFromSuperview];//移除_contentScrollView中的所有视图
        
    }
    
    _showView = showView;//赋值
    
    [_contentScrollView addSubview:showView];//将需要显示的视图添加到_contentScrollView上
    
```

#### 第三步  读取PDF文件
```
//用于读取pdf文件
- (CGPDFDocumentRef)pdfRefByDataByUrl:(NSString *)aFileUrl {
    
    NSData *pdfData;
    
    if ([aFileUrl hasPrefix:@"http://"] || [aFileUrl hasPrefix:@"https://"]) {
     
        pdfData = [NSData dataWithContentsOfURL:[NSURL URLWithString:aFileUrl]];

    }else{
        
        pdfData = [NSData dataWithContentsOfFile:aFileUrl];

    }

    CFDataRef dataRef = (__bridge_retained CFDataRef)(pdfData);

    CGDataProviderRef proRef = CGDataProviderCreateWithCFData(dataRef);
    CGPDFDocumentRef pdfRef = CGPDFDocumentCreateWithProvider(proRef);

    CGDataProviderRelease(proRef);
    
    if (dataRef == nil) {
    
        return nil;
    
    }
    
    CFRelease(dataRef);


    return pdfRef;
    
}
```
#### 第四步  获取所有需要显示的PDF页面
```
//获取所有需要显示的PDF页面

- (void)getDataArrayValue {
    
    size_t totalPages = CGPDFDocumentGetNumberOfPages(_docRef);//获取总页数

    self.totalPage= (int)totalPages;//给全局变量赋值

    NSMutableArray *arr = [NSMutableArray new];
    
    //通过循环创建需要显示的PDF页面，并把这些页面添加到数组中
    
    for(int i =1; i <= totalPages; i++) {

        RiderPDFView *view = [[RiderPDFView alloc] initWithFrame:CGRectMake(0,0,self.view.frame.size.width,self.view.frame.size.height-self.TopHeight) documentRef: _docRef andPageNum:i];

        [arr addObject:view];

    }

    self.dataArray= arr;//给数据数组赋值
    
}
```

#### 第五步  创建RiderPDFView视图,

在RiderPDFView.m文件中绘制视图
```
//重写- (void)drawRect:(CGRect)rect方法

- (void)drawRect:(CGRect)rect {
    
    [self drawPDFIncontext:UIGraphicsGetCurrentContext()];//将当前的上下文环境传递到方法中，用于绘图
    
}

//- (void)drawRect:(CGRect)rect具体的内容

- (void)drawPDFIncontext:(CGContextRef)context {
    
    CGContextTranslateCTM(context,0.0,self.frame.size.height);
    
    CGContextScaleCTM(context,1.0, -1.0);

//上面两句是对环境做一个仿射变换，如果不执行上面两句那么绘制出来的PDF文件会呈倒置效果，第二句的作用是使图形呈正立显示，第一句是调整图形的位置，如不执行绘制的图形会不在视图可见范围内
    
    CGPDFPageRef  pageRef =CGPDFDocumentGetPage(documentRef,pageNum);//获取需要绘制的页码的数据。两个参数，第一个数传递进来的PDF资源数据，第二个是传递进来的需要显示的页码
    
    CGContextSaveGState(context);//记录当前绘制环境，防止多次绘画
    
    CGAffineTransform  pdfTransForm =CGPDFPageGetDrawingTransform(pageRef,kCGPDFCropBox,self.bounds,0,true);//创建一个仿射变换的参数给函数。第一个参数是对应页数据；第二个参数是个枚举值，我每个都试了一下，貌似没什么区别……但是网上看的资料都用的我当前这个，所以就用这个了；第三个参数，是图形绘制的区域，我设置的是当前视图整个区域，如果有需要，自然是可以修改的；第四个是旋转的度数，这里不需要旋转了，所以设置为0；第5个，传递true，会保持长宽比
    
    CGContextConcatCTM(context, pdfTransForm);//把创建的仿射变换参数和上下文环境联系起来
    
    CGContextDrawPDFPage(context, pageRef);//把得到的指定页的PDF数据绘制到视图上
    
    CGContextRestoreGState(context);//恢复图形状态
    
}

```

#### 第六步  加载pdf内容

在PDFReadViewController.m文件中collectionView的代理方法中加载RiderPDFView视图

```
    CollectionViewCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:@"CollectionViewCell" forIndexPath:indexPath];
    
    cell.cellTapDelegate = self;//设置tap事件代理
    
    cell.showView = self.dataArray[indexPath.row];//赋值，设置每个item中显示的内容
    
    return cell;
```

# 最后([Demo](https://github.com/xiaoyeZhang/PDF-READ))

pdf阅读器已经开源了框架,具体使用方法如下

# 第一步：使用CocoaPods导入PDF-READ

CocoaPods 导入

  在文件 Podfile 中加入以下内容：

    pod 'PDF-READ'
  然后在终端中运行以下命令：

    pod install
  或者这个命令：
```
  禁止升级 CocoaPods 的 spec 仓库，否则会卡在 Analyzing dependencies，非常慢
    pod install --verbose --no-repo-update
  或者
    pod update --verbose --no-repo-update
```
  完成后，CocoaPods 会在您的工程根目录下生成一个 .xcworkspace 文件。您需要通过此文件打开您的工程，而不是之前的 .xcodeproj。

# 第二步：初始化PDFReadViewController类,同时传入文档地址

```
    PDFReadViewController *vc = [[PDFReadViewController alloc]init];
    
    vc.fileUrl = @"http://172.16.9.159:8888/002.pdf"; //不能为空,可以为网络地址,也可以为本的存储地址
    
    [self.navigationController pushViewController:vc animated:YES];

    
```

# 第三步（可选）：设置阅读模式,横向,或纵向阅读,是否翻页,设置跳转指定的页码

  ```
  
    vc.scrollDirection = UICollectionViewScrollDirectionHorizontal; // 横向,或纵向阅读
    
    vc.PDFPagingEnabled = YES;  // 是否翻页
    
    vc.item = @"2"; // 指定的页码
    
``` 
