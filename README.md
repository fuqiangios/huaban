# 涂鸦demo（swift）
这是一款涂鸦软件，能够实现对图片的基本操作,供大家参考，主要功能点有：
# 效果图
![(logo)](http://images2015.cnblogs.com/blog/818253/201705/818253-20170526152417013-2009977762.gif)
# 1.截取长图

该功能的主要原理是截取webview的高度所在的区域，所以这种截屏需要在webview加载完毕后获取到webView.scrollView的contensize，然后将webview的高度设置成这个高度再结合上下文进行截屏操作，注意截屏之后webview的尺寸要恢复成之前的尺寸
   
    // 截取webview所有的内容
    func screenShot() -> UIImage {
        var image = UIImage()
        UIGraphicsBeginImageContextWithOptions(self.webView.scrollView.contentSize, true, 0)
        //保存webView当前的偏移量
        let savedContentOffset = self.webView.scrollView.contentOffset
        let saveFrame = self.webView.scrollView.frame`
        
        //将webView的偏移量设置为(0,0)
        self.webView.scrollView.contentOffset = CGPoint(x: 0, y: 0)
        self.webView.frame = CGRect(x: 0, y: 0, width:
        self.webView.scrollView.contentSize.width, height: self.webView.scrollView.contentSize.height)
        
        //在当前上下文中渲染出webView
        self.webView.scrollView.layer.render(in: UIGraphicsGetCurrentContext()!)

        //截取当前上下文生成Image
        image = UIGraphicsGetImageFromCurrentImageContext()!
        
        //恢复webview的偏移量
        self.webView.scrollView.contentOffset = savedContentOffset
        self.webView.frame = saveFrame
        
        UIGraphicsEndImageContext()
        
        return image
    }
    
## 2.画板的封装，能够高高效、方便的实现画板的操作。
这个思路我是参考这篇文章 http://blog.csdn.net/zhangao0086/article/details/43836789  ，他对画板的封装思路很好，大家可以去看一下，不足之处就是图片的缓存搞得有些复杂，我把他的图片缓存逻辑给改了一下。这里主要借鉴他的封装思路。
![(logo)](http://images2015.cnblogs.com/blog/818253/201705/818253-20170526140424622-702145253.png)
## 3.画板的实现   
3.1点击第一个页面的编辑进入画板页面，该页面的主要结构是：底层ScrollView，ScrollView上面放置一个DrawBoard（UIimageView）（放置ScrollView的原因是可以随意缩放图片）

新建DrawBoard，继承自UIimageView，这个DrawBoard就是我们的画板，这里面注意一点：我们绘制的图片不是DrawBoard的image属性，而是backgroundColor。

       scrollView = UIScrollView()
        scrollView.frame = CGRect(x: 0, y: 64, width: KScreenWidth, height: KScreenHeight-50-40-64)
        scrollView.showsHorizontalScrollIndicator = false
        scrollView.showsVerticalScrollIndicator = false
        scrollView.delegate = self
        scrollView.minimumZoomScale = 1
        scrollView.maximumZoomScale = 10
        self.view.addSubview(scrollView!)
        
        drawBoardImageView = DrawBoard.init(frame:scrollView.bounds)
        drawBoardImageView.isUserInteractionEnabled = true
        // 对长图压缩处理
        let scaleImage = UIImage.scaleImage(image: self.editorImage, scaleSize: scrollView.cl_height/self.editorImage.size.height)
        drawBoardImageView.backgroundColor = UIColor(patternImage: scaleImage)
        scrollView?.addSubview(drawBoardImageView)
3.2 因为截取的是一个长图，所以如果直接设置 drawBoardImageView.backgroundColor = UIColor(patternImage: self.editorImage)，就会只显示图片的一部分，所以要对图片进行压缩
。在压缩图片时需要注意一点：图片的大小还是屏幕大小，只是内容压缩，如果图片宽度小于屏幕宽度，图片会平铺铺满整个界面。下面是压缩图片的代码

	    static func scaleImage(image: UIImage) -> UIImage {
        // 画板高度
        let boardH = KScreenHeight-64-50-40
        // 图片大小   UIScreen.main.scale屏幕密度，不加这个图片会不清晰
        UIGraphicsBeginImageContextWithOptions(CGSize(width:KScreenWidth,height:boardH), false, UIScreen.main.scale)
        // 真正图片显示的位置
        // 图片的宽高比
        let picBili: CGFloat = image.size.width/image.size.height
        // 画板的宽高比
        let boardBili: CGFloat = KScreenWidth/boardH
        
        // 如果图片太长，以高为准,否则以宽为准
        if picBili<=boardBili {
            image.draw(in: CGRect(x: 0.5*(KScreenWidth-boardH*picBili), y: 0, width: boardH*picBili, height: boardH))
        } else {
            image.draw(in: CGRect(x: 0, y:0.5*(boardH-KScreenWidth/picBili) , width: KScreenWidth, height: KScreenWidth/picBili))
        }
        
        let scaledImage = UIGraphicsGetImageFromCurrentImageContext()
        
        UIGraphicsEndImageContext()
        
        //对图片包得大小进行压缩
        let imageData =  UIImagePNGRepresentation(scaledImage!)
        let m_selectImage = UIImage.init(data: imageData!)
        return m_selectImage!
    }
    
    
3.2 画板的使用和结构
![(logo)](http://images2015.cnblogs.com/blog/818253/201705/818253-20170526145227450-1786765220.png)

## 4.马赛克功能  
![(logo)](http://images2015.cnblogs.com/blog/818253/201705/818253-20170526150905935-1737313357.png)

原理：先将要绘制的图片转换成马赛克图片，然后将马赛克图片设置成画笔的颜色就完美实现了马赛克功能。

具体参考demo    
## 5.文本输入功能
思路：手指触摸屏幕时，在屏幕上绘制一个textView，当textView输入结束时，将文字和图片绘制到同一个image中。 绘制的文字同样也支持撤销，橡皮擦等操作

![(logo)](http://images2015.cnblogs.com/blog/818253/201705/818253-20170526150404450-806964848.gif
)

## 6.DrawBoard类中图片的撤销与前进功能  

6.1在DrawBoard类中有一个image的管理类DBUndoManager，主要用于管理我们绘制了多少图片，控制撤销操作和前进操作

6.2DBUndoManager类中的imageArray属性保存着我们每次绘制结束后的image（包括画笔的绘制后的image，文本的输入后的image）

6.3 imageArray存储方法：当用户开始绘制时，先判断是不是最开始的那张图片（没有做任何操作的图片），如果是的，就将imageArray清空，如果不是最初的那张原图，就继续把绘制的image加入到数组中，这里面的逻辑也不复杂，需要认真的理一下。所以，在这里就不多少了，具体看demo，思考！
## 7.图片的保存  
	    
	    // 返回画板上的图片，用于保存
    func takeImage() -> UIImage {
        UIGraphicsBeginImageContext(self.bounds.size)
        
        self.backgroundColor?.setFill()
        UIRectFill(self.bounds)
        
        self.image?.draw(in: self.bounds)
        
        let image = UIGraphicsGetImageFromCurrentImageContext()
        UIGraphicsEndImageContext()
        
        return image!
    }
    
    //MARK: - 下载图片
    func clickLoadBtn(){
        let alertController = UIAlertController(title: "提示", message: "您确定要保存整个图片到相册吗？", preferredStyle: .alert)
        let cancelAction = UIAlertAction(title: "取消", style: .cancel, handler: nil)
        let okAction = UIAlertAction(title: "确定", style: .default, handler: {
            action in

            UIImageWriteToSavedPhotosAlbum(self.drawBoardImageView.takeImage(), self, #selector(EditorViewController.image(_:didFinishSavingWithError:contextInfo:)), nil)
        })
        alertController.addAction(cancelAction)
        alertController.addAction(okAction)
        self.present(alertController, animated: true, completion: nil)
    }
    // 保存图片的结果
    func image(_ image: UIImage, didFinishSavingWithError error: NSError?, contextInfo:UnsafeRawPointer) {
        if let err = error {
            UIAlertView(title: "错误", message: err.localizedDescription, delegate: nil, cancelButtonTitle: "确定").show()
        } else {
            UIAlertView(title: "提示", message: "保存成功", delegate: nil, cancelButtonTitle: "确定").show()
        }
    }

个人邮箱：aliang_chenchen@163.com