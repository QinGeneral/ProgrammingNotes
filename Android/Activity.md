# Android 四大组件之 Activity

[TOC]

## Activity生命周期

![ Activity生命周期](https://blog-pic-1251295613.cos.ap-guangzhou.myqcloud.com/1675412576.751284SmartPic.png)

- onStart、onStop 代表着应用是否可见  
- onResume、onPause 代表应用是否在前台

- onSaveInstanceState（onStop之前调用，和onPause没有必然的先后顺序）  
- onRestoreInstanceState（onStart之后调用，和onResume没有必然的先后顺序）  
以上两个方法调用条件：
- 应用被杀死  
- 配置改变（比如手机方向，添加android:configChanged属性后不会触发，会调用onConfiguration函数）  
- Home键、启动新Activity（单独触发onSaveInstanceState）

### 常见事件生命周期

- 启动 A：onCreate -> onStart -> onResume  
- A 启动 B：A.onPause -> A.onStop （如果B采用透明主题，则A.onStop不会调用）  
- 启动 B 后，返回 A：onRestart -> onStart -> onResume  
- 返回键：onPause -> onStop -> onDestroy  
- A 启动 B，A 的 onPause 执行之后，B才 启动。在 onPause 之中做轻量级工作，能加快 B 的启动速度


## Activity 启动模式

四种模式可根据字面意思理解，不过还要注意一些细节

- standard 标准模式：
  - 每次启动会创建一个新的Activity
  - A启动B，B会位于A的栈中
  - 默认的启动模式
- singleTop 栈顶复用模式
  - 要启动的Activity在栈顶则直接使用，不创建新的Activity
  - 第二次启动在栈顶，会调用onNewIntent、onResume方法，onCreate、onStart不会调用
- singleTask 栈内复用模式
  - 要启动的Activity在栈内则直接使用，不创建新的Activity
  - 第二次启动在栈顶，会调用onNewIntent、onResume方法，onCreate、onStart不会调用
  - 可通过TaskAffinity属性指定要启动的Activity将位于的栈名
  - 具有clearTop效果：在栈内，要启动的A之上有B、C，会让B、C出栈，然后复用A
- singleInstance 单一实例模式
  - 要启动的Activity会新建一个栈，并且此Activity会独占这个栈
  
> 注：可以使用 adb shell dumpsys Activity 查看Activity栈信息，来分析Activity启动时栈的情况

### 应用场景

- standard：Activity默认模式，一般Activity都用这个；
- singleTop：当外界多次跳转到一个页面是可以使用这个模式，比如从一些下拉栏通知界面点击进入一个页面的情景，避免了因为多次启动导致的需要返回多次的情况；
- singleTask：可用于应用的主界面，比如浏览器主页，外界多次启动时不会受子页面干扰，clearTop效果也会清楚主页面之上的页面；
- singleInstance：可用于和程序分离的页面，比如通话页面、闹铃提醒页面。

> 注：在A -> B -> C 时，B不要采用singleInstance，否则，退出再打开时，会是B页面（此属性未验证）

## IntentFilter 匹配规则

Intent隐式启动的三个属性：action、category、data。

对于这三个属性的匹配规则如下：
- action：代码中有一个及以上与“xml过滤规则”中的相同即可；
- category：代码中所有的必须与“xml过滤规则”中的相同；
- data：同 action。

> 注：  
- 代码中隐式启动时，会默认添加android.intent.category.DEFAULT，所以xml必须含有此属性才能隐式启动；
- Service尽量采用显示启动。