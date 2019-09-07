## CAShapeLayer

### 属性解释

#### strokeStart和strokeEnd

描边的起点和终点，取值范围是0-1，strokeStart默认是0，strokeEnd默认是1，常会用`strokeEnd`做动画，来绘制一些图形，比如进度条。

#### lineCap 

终点的样式

<img src="https://github.com/lvchunf/chunfNotes/blob/master/resources/WechatIMG5.png"/>


#### lineJoin 

拐点的样式。

<img src="https://github.com/lvchunf/chunfNotes/blob/master/resources/WechatIMG4.jpeg"/>

#### lineDashPattern

边线的样式，是一个数组，奇数表示实线的长度，偶数表示空白的长度，例如`[5,2,8,3]`,表示实线长度为5，空白长度2，接着实线长度为8，空白长度为3，最后会按照这个规律一直循环，直到storkeEnd所在的点。

#### fillRule

填充原则，填充肯定是要填充路径内部的，那么如何判断为路径内部呢，简单的直接用肉眼就直接能看出来，复杂的其实是有一套规则的，这个规则可以参考 [这个链接](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Attribute/fill-rule)

- `evenOdd` 奇偶性，表示一个点与所有路径相交的个数如果是奇数则判断在路径内，如果是偶数则判断在路径外。

- `nonZero` 一个点向左向右与路径相交点的个数正好抵消，则判断为在路径内。

<img src="https://github.com/lvchunf/chunfNotes/blob/master/resources/WechatIMG6.png"/>

### 利用strokeEnd 做进度条动画

```
 // 创建CAShapeLayer
let shapeLayer = CAShapeLayer()
shapeLayer.bounds = CGRect(x: 0, y: 0, width: 100, height: 100)
shapeLayer.fillColor = UIColor.clear.cgColor
shapeLayer.strokeColor = UIColor.red.cgColor
shapeLayer.lineWidth = 2.0

// 添加路径
let path = UIBezierPath(ovalIn: CGRect(x: 0, y: 0, width: 100, height: 100))
shapeLayer.path = path.cgPath

shapeLayer.position = view.center
view.layer.addSublayer(shapeLayer)

// 添加动画
let animation = CABasicAnimation(keyPath: "strokeEnd")
animation.duration = 2.0
animation.timingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseOut)
animation.fromValue = 0.0
animation.toValue = 1.0
animation.fillMode = kCAFillModeForwards
animation.isRemovedOnCompletion = false
shapeLayer.add(animation, forKey: nil)

```
