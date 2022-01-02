# 信息无障碍-iOS
![中国无障碍信息联盟](/images/accessibilitylogo.png)
## 写在前面
### 什么是无障碍
>信息无障碍是指无论健全人还是残疾人、无论年轻人还是老年人都能从信息技术中获益，任何人在任何情况下都能平等地、方便地、无障碍地获取信息、利用信息。
>主要用于互联网环境，大意是：互联网产品通过进行易用性、可用性等优化，可以被老年人、视障者、听障者、读写障碍人士等用户顺畅使用，同时可以更高效、更便捷地被所有用户使用。  

概括一句话：无障碍是指一个应用里的内容能够被清晰完整的朗读出来当视障用户触摸到某个视图上时，并且在用户进行某些操作时，能够及时并且准确的给到用户反馈。
### iOS无障碍体验
上面给出了无障碍的定义，那它具体的表现形式是什么样子的呢，我们来站在iOS系统的用户视角体现一下。  
VoiceOver是苹果公司实现的一种内置屏幕阅读程序，能够将屏幕上显示的内容朗读出来，满足视障人士的使用需求，iOS的无障碍适配依赖于VoiceOver。  
VoiceOver能够读出屏幕上的无障碍元素所提供的信息，对于UILabel，UIButton这类的标准控件，其本身就是支持无障碍的，默认的无障碍信息即可满足用户需求，但是相对比较复杂的视图层级和自定义的控件就需要我们在编写程序的时候为我们的无障碍元素提供准确，易懂的信息供VoiceOver朗读。
#### 快速开关VoiceOver
想要快速开启和关闭 VoiceOver 除了对着万能的 Siri 说打开或者关闭 VoiceOver 外，还有一种方便的办法。进入 设置 > 辅助功能，滚动到最底下，点开辅助功能快捷键并勾选旁白。之后在任何情况下都只需要连按三次 Home 键/电源键就可以快速开启和关闭 VoiceOver 了。  
#### VoiceOver手势
| 操作  | 效果 |
| ---  | ---- |
| 单击  | 定位焦点到点击位置，并开始朗读焦点内容 |
|双击   | 选定操作，效果同正常模式下单击 |
|双指Z字 | 返回上一层级（详情页->列表页）|
|三指滚动| 向上或向下滑动列表 |
|单指下扫| 朗读焦点中的下个元素 |
|单指上扫| 朗读焦点中的上个元素 |  

#### 转子设置
单指上下扫动所读取的内容取决于转子当前所在的选项，我们可以在设置->辅助功能->旁白->转子，这个路径里面去设置转子可以显示的选项（一般会默认选择常用的几项，基本可以满足日常操作）。  
用户可以使用双指同时分别上下滑动的操作选择无障碍转子的不同选项，当选定了某一选项后，单指上滑或者单指下滑VoiceOver就会朗读对应的内容。  
## 我们为什么要做无障碍
### 用户需求
*以下数据来自[中国互联网视障用户基本情况报告](https://www.siaa.org.cn/static/image/img_media/%E4%B8%AD%E5%9B%BD%E4%BA%92%E8%81%94%E7%BD%91%E8%A7%86%E9%9A%9C%E7%94%A8%E6%88%B7%E5%9F%BA%E6%9C%AC%E6%83%85%E5%86%B5%E6%8A%A5%E5%91%8A.pdf)*  
2019年的数据：中国目前有1700多万视障人士，智能手机用户占比92%，其中21-50岁之间的人士占比80%，有比较强烈的移动应用使用诉求。但是在这些视障用户使用移动应用的过程中，满意度整体较低，非常满意的只有18%。  
这表示我们的移动应用在无障碍适配方面的工作做得不够好，远不能满足视障人群的要求，因此适配信息无障碍变得尤为重要。
![满意度](/images/accessibilityPic01.png)  
视障者对读屏功能的依赖情况，有 83%的视障者在操作手机、电脑的时候是完全依赖读屏功能的，14%的视障者是用眼睛看结合着读屏功能操作手机、电脑的。可以看出，读屏软件已成为视障者的“眼睛”。  
![依赖度](/images/accessibilityPic02.png)  
视障者日常上网的需求，包括社交（包括聊天、逛论坛、刷微博等）、看新闻、看书、 听音乐、玩游戏、购物等已成为视障者日常上网主要做的事情。  
![使用内容](/images/accessibilityPic03.png)  
### 国家政策
国务院：[国务院关于印发“十四五”残疾人保障和发展规划的通知](https://mp.weixin.qq.com/s/ynfmEvHZpQo2XfBUlE9jSA)  
>促进信息无障碍国家标准推广应用，加强对互联网内容可访问性的测试、认证能力建设，开展互联网和移动互联网无障碍化评级评价。  

工信部：[工业和信息化部关于印发《互联网应用适老化及无障碍改造专项行动方案》的通知](https://www.miit.gov.cn/zwgk/zcwj/wjfb/txy/art/2020/art_3e1c2bf3f1d6410fab42728a33ec7c3b.html)  
>将信息无障碍情况纳入企业信用评价，首批适老化及无障碍改造APP名单中包含今日头条，抖音，火山小视频。  

## iOS无障碍适配
### 编程接口
无障碍编程接口由两个非正式协议、一个类、以及少数常量构成。  
+ __UIAccessibility 非正式协议：__ 实现 UIAccessibility 协议的对象， 可以报告无障碍状态（即他们是否是可访问的），并且提供其自身的描述信息。标准 UIKit 控件和视图已默认实现了 UIAccessibility 协议。  
+ __UIAccessibilityContainer 非正式协议：__ 此协议允许UIView的子类作为 分离元素，具有无障碍特性构建部分或者全部对象。当对象包含在视图中，其本身又不是UIView的子类，导致不能被自动地无障碍访问，此时，该协议非常有用。  
+ __UIAccessibilityElement 类：__ 这个类定义了一个能够通过UIAccessibilityContainer协议返回的对象。当一个元素不是自动地支持无障碍访问，可以创建一个IAccessibilityElement实例来展示它， 例如一个非UIView继承对象，或者一个不存在的对象。  
+ __UIAccessibilityConstants.h 头文件：__ 这个头文件定义了某些常量， 这些常量既可以描述一个可展示的无障碍元素的特征，也可以描述应用发布的通知。  
### UIAccessibility非正式协议
UIAccessibility非正式协议提供关于应用用户界面的无障碍信息，辅助应用（voiceOver）将其提供的信息传递给残障用户，帮助其使用应用。  
标准UIKit控件和视图都默认实现了UIAccessibility的方法，所以VoiceOver默认可以访问。不过当视图层级比较复杂的时候默认值可能不太完整，所以我们需要在有必要的时候给应用中无障碍信息缺失的一些视图控件去补充完整，可读的信息。  

```Objective-C
//   UIKit/UIAccessibility.h
/*
 UIAccessibility
 
 UIAccessibility is implemented on all standard UIKit views and controls so
 that assistive applications can present them to users with disabilities.
 
 Custom items in a user interface should override aspects of UIAccessibility
 to supply details where the default value is incomplete.
 
 For example, a UIImageView subclass may need to override accessibilityLabel,
 but it does not need to override accessibilityFrame.
 
 A completely custom subclass of UIView might need to override all of the
 UIAccessibility methods except accessibilityFrame.
 */
@interface NSObject (UIAccessibility)
@property (nonatomic) BOOL isAccessibilityElement;
@property (nullable, nonatomic, copy) NSString *accessibilityLabel;
@property (nullable, nonatomic, copy) NSString *accessibilityHint;
@property (nullable, nonatomic, copy) NSString *accessibilityValue;
@property (nonatomic) UIAccessibilityTraits accessibilityTraits;
@end
```  

### 常用属性
#### 基础属性
+ __isAccessibilityElement：__ 标识当前组件是否是无障碍组件，UIKit控件中其默认值为YES。通常情况下当其值为YES时才被VoiceOver当做一个无障碍的元素。  
> + 该方式的唯一例外是该视图只是作为其他应该被可访问项目的容器。这样的视图应该实现 UIAccessibilityContainer 协议并将该属性设置为 NO。
> + UITableViewCell本身被设置为YES或者NO都能获取到其焦点，但是当我们还想要获取cell上面子控件的焦点时，就需要将cell本身的isAccessibilityElement属性设置为NO。
+ __accessibilityLabel：__ 简洁易懂的标签，标识该无障碍元素的作用。当接收者是一个UIKit控件的时候，该属性的值默认从控件的标题上取（label读label.text，button读button.title）. 
+ __accessibilityHint：__ 简单描述无障碍元素的执行结果。当accessibilityLabel不能清除的传达结果的时候，accessibilityHint能够帮助用户理解在无障碍元素上操作会发生什么。  
+ __accessibilityValue：__  设置无障碍属性的值。多用于当前无障碍元素的标签是固定的，但值会有多种的情况，例如，一个呈现文本域的无障碍元素有标签“Message”，但该无障碍元素的值是当前文本域中的文本。  
+ __accessibilityTraits：__  标识当前无障碍元素的类型或特征，可以由多种类型组合。  
```Objective-C
typedef uint64_t UIAccessibilityTraits NS_TYPED_ENUM;

// Used when the element has no traits.
UIKIT_EXTERN UIAccessibilityTraits UIAccessibilityTraitNone;

// Used when the element should be treated as a button.
UIKIT_EXTERN UIAccessibilityTraits UIAccessibilityTraitButton;

// Used when the element should be treated as a link.
UIKIT_EXTERN UIAccessibilityTraits UIAccessibilityTraitLink;

// Used when an element acts as a header for a content section (e.g. the title of a navigation bar).
UIKIT_EXTERN UIAccessibilityTraits UIAccessibilityTraitHeader API_AVAILABLE(ios(6.0));

// Used when the text field element should also be treated as a search field.
UIKIT_EXTERN UIAccessibilityTraits UIAccessibilityTraitSearchField;

// Used when the element should be treated as an image. Can be combined with button or link, for example.
UIKIT_EXTERN UIAccessibilityTraits UIAccessibilityTraitImage;
```  
+ __accessibilityElementsHidden：__  标识当前无障碍元素的无障碍属性是否被隐藏，默认值是NO。当我们将一个控件的该属性设置为YES时，可以正常展示但辅助应用VoiceOver无法获取到其焦点。  
#### 进阶适配
当我们熟练掌握上面这几种属性的含义并且能灵活运用与不同业务场景的时候，我们已经可以完成绝大多数的无障碍适配工作，能够反馈给视障用户完整清晰简洁的无障碍信息了，但是当业务场景足够复杂的时候，只靠上面这些属性远远做不到完美适配。下面我们看一些进阶的用法。  
##### UIAccessibilityNotification
__UIAccessibilityLayoutChangedNotification：__ 当界面上的元素UI布局变化，比如某些控件消失或展示时，由应用发出的一个通知。包含一个参数，该参数是一个需要VoiceOver朗读的字符串或者是无障碍焦点需要转移到的元素对象。  
__UIAccessibilityScreenChangedNotification：__  当一个新的视图出现，并覆盖屏幕的主要部分时，由应用发送的通知，参数和UIAccessibilityLayoutChangedNotification的参数相同。  
__UIAccessibilityVoiceOverStatusChanged：__ 当VoiceOver打开或者关闭时，系统发出的通知。可用于某些在VoiceOver开启或者关闭的情况下的特定逻辑的开关控制。  
__UIAccessibilityPageScrolledNotification：__ 列表滚动操作完成后由应用发出的通知。参数是一个字符串，该字符串应该表示当前的列表位置，如果连续收到相同参数的该通知，那么VoiceOver会告诉用户已经到达列表边缘。  
##### UIAccessibilityAction
__- (BOOL)accessibilityActivate：__ 当无障碍用户双击选定某个元素时，该方法会被调用，一般用来在该方法中实现某些比较难以在无障碍模式下打开的功能。比如我们的某个按钮的点击需要统计无障碍模式和正常模式的埋点，便可以在该方法中进行上报，其实我们可以在该方法中实现双击后想要做的任何事。  
__- (BOOL)accessibilityPerformMagicTap：__ 用户触发魔法轻拍（双指双击屏幕）动作时，会执行该函数。一般情况下该方法中实现一个APP最重要的状态转换，比如相机应用中的拍照动作或者音乐APP中可以在该方法中实现音乐的暂停和播放动作。  
需要注意的是：Magic Tap在你的App中应该只在AppDelegate文件中实现一次，这样才满足Magic Tap的设定和意义。  
__- (BOOL)accessibilityScroll:(UIAccessibilityScrollDirection)direction：__ 当用户使用三指滑动手势滚动列表时会触发该函数。  
```Objective-C
/*
 If the user interface requires a scrolling action (e.g. turning the page of a book), a view in the view 
 hierarchy should implement the following method. The return result indicates whether the action 
 succeeded for that direction. If the action failed, the method will be called on a view higher 
 in the hierarchy. If the action succeeds, UIAccessibilityPageScrolledNotification must be posted after
 the scrolling completes.
 default == NO
 */
typedef NS_ENUM(NSInteger, UIAccessibilityScrollDirection) {
  UIAccessibilityScrollDirectionRight = 1,
  UIAccessibilityScrollDirectionLeft,
  UIAccessibilityScrollDirectionUp,
  UIAccessibilityScrollDirectionDown,
  UIAccessibilityScrollDirectionNext API_AVAILABLE(ios(5.0)),
  UIAccessibilityScrollDirectionPrevious API_AVAILABLE(ios(5.0)),
};
```  
## 总结
无障碍的适配工作总体来说技术难度较小，只需要实现一些系统接口即可做到。但是我们要知道从做到到做好，之间还需要的一个关键环节，是作为开发人员的社会责任感。我们应当从如何真正使我们的APP便利于视障者的使用的角度出发去思考每一个无障碍bug的适配，努力做到信息准确，简洁易懂，最大程度方便视障用户的使用。
