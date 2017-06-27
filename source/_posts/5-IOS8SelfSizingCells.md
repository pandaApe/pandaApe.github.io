title: iOS8后使用AutoLayot自动计算Cell高度
date: 2015-12-03 22:48:34
categories: iOS
---
iOS8中引入了新功能，用较少的代码实现了Cell自动计算高度。

<!--more-->
## 概述
在iOS8中，Apple 的UITableView引入了一个新功能，Self-Sizing Cells。这个新东西对开发有什么好处呢？我们知道，在IOS8以前，如果显示一个动态高度的tableview cell，你必须在 tableView:heightForRowAtIndexPath:方法中手动计算Row的高度。现在iOS8中的Self-Sizing Cell 提供一个计算动态高度的解决方案.

## 使用Self-Sizing Cells 的正确姿势
要满足下面三个条件：
* Tableview 的cell 要使用 autolayout 约束
* 指定Tableview的estimatedRowHeight属性的值
* 设置Tableview的rowHeight为UITableviewAutomaticDimension

用代码表示后面两点就是这样：

```swift
self.tableview.estimatedRowHeight = 40
self.tableview.rowHeight = UITableviewAutomaticDimension
```

仅仅用AutoLayout+两行代码，你再也不需要实现
``` swift
func tableView(tableView: UITableView, estimatedHeightForRowAtIndexPath indexPath: NSIndexPath) -> CGFloat 
func tableView(tableView: UITableView, heightForRowAtIndexPath indexPath: NSIndexPath) -> CGFloat
```

## Demonstrate It!
1.在Storyboard 上拖一个UITableviewController在Tableview 中我们加入一个自定义的cell, cell 中有用户头像ImageView，名字Label，和地址Label,把地址Label的lines值设置为0，让label高度可以随内容自动增加。
![](http://ww4.sinaimg.cn/large/8c0dc373gw1eyp4bg4fbbj21go0v410b.jpg)
2.添加Autolayout约束，这一步相当重要。我这里使用了固定底部间距，使Superview能有确定的高度。
![](http://ww4.sinaimg.cn/large/8c0dc373gw1eyp4a8r42vj21kw0mvq9y.jpg)
  
分析：头像UIImageview顶部距Superview的固定，名字UILabel和imageview的顶部Margin相等，名字UILabel的高度固定，地址Label的高度和底部Margin固定，因此superview的高度就能根据子View高度的变化而变化了。那么在这个Demo中，地址的Label内容越多，高度值就会越大，从而Cell的调度变会越大。
3.代码：
```swift
class ViewController: UITableViewController {
    
    var dataArray:[(avatar:UIImage,userName:String,address:String)]!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        dataArray = Array()
        
        for i in 1...10{
            
            var addressStr = ""
            for _ in 0...i{
                addressStr += "Apple Inc. 1 Infinite Loop Cupertino, CA 95014 408-996-1010. "
            }
            let item = (UIImage(named: "\(i%6)")!,"User-\(i)",addressStr)
            dataArray.append(item)
        }
        
        self.tableView.rowHeight = UITableViewAutomaticDimension
        self.tableView.estimatedRowHeight = 70
    }
    
    override func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
        
        let dataItem = dataArray[indexPath.row]
        
        let cell = tableView.dequeueReusableCellWithIdentifier("msgCell", forIndexPath: indexPath)
        let ivAvatar = cell.viewWithTag(100) as! UIImageView
        let lbUserName = cell.viewWithTag(101) as! UILabel
        let lbAddress = cell.viewWithTag(102) as! UILabel
        
        ivAvatar.image = dataItem.avatar
        lbUserName.text = dataItem.userName
        lbAddress.text = dataItem.address
        
        return cell
    }
    
    override func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {      
        return dataArray.count
    }
}
```
搞定：
![](http://ww4.sinaimg.cn/large/8c0dc373gw1eyp4e54wfdg208g0f34qp.gif)

## 参考资料
[Working with Self-Sizing Table View Cells](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/WorkingwithSelf-SizingTableViewCells.html)