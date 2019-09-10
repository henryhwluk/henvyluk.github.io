---

title: CustomTabBar
urlPath: CustomTabBar
date: 2019-06-13
updated: 2019-06-13
tag: [iOS, TabBar, iPhone X]

---

### CustomTabBarController

* viewDidLoad

```swift
override func viewDidLoad() {
        super.viewDidLoad()
        
        let tabBar = CustomTabBar()
        tabBar.delegate = self
        self.setValue(tabBar, forKeyPath: "tabBar") 
        
        let vc1 = ViewController()
        let nvc1: UINavigationController = NavigationViewController(rootViewController: vc1)
        ...
        ...
             
        
        
        let normalDict: NSDictionary = [NSAttributedStringKey.foregroundColor: UIColor.white, NSAttributedStringKey.font : UIFont.boldSystemFont(ofSize: 10)]
        let selectedDict: NSDictionary = [NSAttributedStringKey.foregroundColor: UIColor.white, NSAttributedStringKey.font : UIFont.boldSystemFont(ofSize: 10)]

        let tabbarItem1 = UITabBarItem(title: NSLocalizedString("xx", comment: ""), image: UIImage(named: "xx.png"), selectedImage: UIImage(named: "xx.png"))
        psdetailTabbar.setTitleTextAttributes((normalDict as! [NSAttributedStringKey : Any]), for: .normal)
        psdetailTabbar.setTitleTextAttributes((selectedDict as! [NSAttributedStringKey : Any]), for: .selected)
        ...
        ...
        
        
        
        nvc1.tabBarItem = tabbarItem1
        ...
        ...
        
        
        
        self.viewControllers = [nvc1,...]
        
        for vc in (self.viewControllers?.enumerated())! {
            let nvc = vc.element as! NavigationViewController
            nvc.navigationBar.barTintColor = UIColor.white
            nvc.viewControllers.first?.view.backgroundColor = UIColor.white
            let bgView = UIImageView(frame: UIScreen.main.bounds)
            bgView.image = UIImage(named: "home_bg")
            nvc.viewControllers.first?.view.addSubview(bgView)
        }
        
        for item in (self.tabBar.items?.enumerated())! {
            //item.element.imageInsets = UIEdgeInsetsMake(7, 0, -7, 0)
            item.element.image = item.element.image?.withRenderingMode(.alwaysOriginal)
            item.element.selectedImage = item.element.selectedImage?.withRenderingMode(.alwaysOriginal)
            
        }
  }
```

<!-- more -->

### CustomTabBar

* init

```swift
override init(frame: CGRect) {
        super.init(frame: frame)
        let btn = CustomButton()
        self.addSubview(btn)
        btn.addTarget(self, action: #selector(clickPublish), for: .touchUpInside)
        self.publishButton = btn
        self.barTintColor = UIColor.navBgColor()
}
```

* layoutSubviews

```swift
  override func layoutSubviews() {
        super.layoutSubviews()

        let buttonW = barWidth / 5
        let buttonH = barHeight
        let buttonY = 0
        var buttonIndex: CGFloat = 0
        
        self.publishButton.frame = CGRect(x: 0, y: 0, width: buttonW, height: buttonH)
        self.publishButton.center = CGPoint(x: buttonW * 0.5, y: barHeight * 0.5)
        
        for view in self.subviews.enumerated() {
            if !view.element.isKind(of: NSClassFromString("UITabBarButton")!){
                continue
            }
            let buttonX = buttonW * ((buttonIndex >= 0) ? (buttonIndex + 1) : buttonIndex)
            if UIDevice.current.modelName == "iPhone X" || UIDevice.current.modelName == "iPhone XS" || UIDevice.current.modelName == "iPhone XS Max" || UIDevice.current.modelName == "iPhone XR" {
                view.element.frame = CGRect(x: buttonX, y: CGFloat(buttonY), width: buttonW, height: 49)
            }else {
                view.element.frame = CGRect(x: buttonX, y: CGFloat(buttonY), width: buttonW, height: buttonH)
            }
            buttonIndex += 1
        }
    }
```
### CustomButton

* init

```swift
override init(frame: CGRect) {
        super.init(frame: frame)
        self.titleLabel?.textAlignment = .center
        self.titleLabel?.font = UIFont.boldSystemFont(ofSize: 10)
        self.setImage(UIImage(named: "tabbar_home_unselected"), for: .normal)
        self.setTitle(NSLocalizedString("xx", comment: ""), for: .normal)
        self.setTitleColor(UIColor.shadowTextColor(), for: .normal)
    }
```
* titleRect

```swift
override func titleRect(forContentRect contentRect: CGRect) -> CGRect {
        let space = CGFloat(2)
        
        let titleW = contentRect.width
        let titleH = CGFloat(12)
        let titleX = 0
        let titleY = CGFloat(49) - space - titleH
        return CGRect(x: CGFloat(titleX), y: titleY, width: titleW, height: titleH)
    }
```

* imageRect

```swift
override func imageRect(forContentRect contentRect: CGRect) -> CGRect {
        let space = CGFloat(5)
        
        let imageH = CGFloat(24.5)
        let imageW = imageH
        let imageX = (contentRect.width - imageH)/2
        let imageY = space
        return CGRect(x: imageX, y: imageY, width: imageW, height: imageH)
    }
```





