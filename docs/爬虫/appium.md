# 安装&启动

需要 Android 开发环境，推荐直接下载 Android Studio，下载 sdk 的时候需要翻墙，然后配置环境变量

```
ANDROID_HOME="/home/jianxin/Android/Sdk"
PATH="$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools"
JAVA_HOME="/opt/android-studio/jre"
PATH="$PATH:$JAVA_HOME/bin"

export PATH
```

安装 appium

```
npm install -g appium
```

> appium-desktop [下载](https://github.com/appium/appium-desktop)

!> 因为 linux 下安装 appium-desktop 可能会出错，比较麻烦，推荐在 windows 下安装 appium-desktop，然后去连接 linux 下的 appium 服务。

启动 appium

```
appium
```

安装 python 操作 appium 的客户端

```
pipenv install Appium-Python-Client
```

Android 手机连上电脑， 用 adb 查看连接情况，手机型号等信息

```
adb devices -l
```

结果

```
List of devices attached
16a4fc1c               device usb:1-1 product:msm8916_32 model:YQ601 device:msm8916_32 transport_id:1
```

`model` 后显示的就是手机型号

下面看官方的例子

``` python
# Android environment
import unittest
from appium import webdriver

desired_caps = {
    'platformName': 'Android',
    'deviceName': 'YQ601',
    'appPackage': 'com.tencent.mm',
    'appActivity': '.ui.LauncherUI'
    # 'platformVersion': '4.2',
    # 'app': '../../../apps/selendroid-test-app.apk', 如果没有安装，可以直接指定 apk
}

driver = webdriver.Remote('http://localhost:4723/wd/hub', desired_caps)
```

> 更多 Desired 信息看[官方文档](http://appium.io/docs/en/writing-running-appium/caps/index.html)

运行这段脚本，手机就会自动打开微信登陆页了

再看个例子

自动点击登陆，输入手机号

``` python
import unittest
from appium import webdriver
from selenium.webdriver.support.wait import WebDriverWait
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC

desired_caps = {
    'platformName': 'Android',
    'deviceName': 'YQ601',
    'appPackage': 'com.tencent.mm', # 可以在 AndroidManifest.xml 中找到
    'appActivity': '.ui.LauncherUI' # 可以在 AndroidManifest.xml 中找到
    # 'platformVersion': '4.2',
    # 'app': '../../../apps/selendroid-test-app.apk', 如果没有安装，可以直接指定 apk
}

driver = webdriver.Remote('http://localhost:4723/wd/hub', desired_caps)
wait = WebDriverWait(driver, 30)
login = wait.until(EC.presence_of_element_located((By.ID, 'com.tencent.mm:id/d37')))
login.click()
phone = wait.until(EC.presence_of_element_located((By.ID, 'com.tencent.mm:id/ht')))
phone.set_text('18888888888')
```

# appium-desktop 使用

1、

![](http://os6ycxx7w.bkt.clouddn.com/images/20180704233627.png)

2、

![](http://os6ycxx7w.bkt.clouddn.com/images/20180704233846.png)

3、

![](http://os6ycxx7w.bkt.clouddn.com/20180704234520.png)

# API

## 查找元素

``` python
login = driver.find_element_by_id('com.tencent.mm:id/d37')
# 或者
login = wait.until(EC.presence_of_element_located((By.ID, 'com.tencent.mm:id/d37')))
```

## 点击

`def tap(self, positions, duration=None):`

* positions: 点击的位置组成的列表，最多五个手指（五个点）
* duration: 点击的持续时间。毫秒

举个栗子

``` python
driver.tap([(100, 20), (100, 60), (100, 100)], 500)
```

对于元素，可以直接使用 `click()` 方法

## 屏幕拖动

`def scroll(self, origin_el, destination_el):`

从元素 origin_el 滚动到元素 destination_el

* origin_el: 被操作的元素
* destination_el: 目标元素

``` python
driver.scroll(el1, el2)
```

还可以使用 `swipe()` 模拟从 A 点滑到 B 点

`def swipe(self, start_x, start_y, end_x, end_y, duration=None):`

* start_x: 开始位置的横坐标
* start_y: 开始位置的纵坐标
* end_x: 终止位置的横坐标
* end_y: 终止位置的纵坐标
* duration: 持续时间，毫秒

``` python
driver.swipe(100, 100, 100, 400, 5000)
```

还可以使用 `flick()` 快速从 A 点滑动到 B 点

`def flick(self, start_x, start_y, end_x, end_y):`

* start_x: 开始位置的横坐标
* start_y: 开始位置的纵坐标
* end_x: 终止位置的横坐标
* end_y: 终止位置的纵坐标

``` python
driver.flick(100, 100, 100, 400)
```

## 拖拽

可以使用 `drag_and_drop()` 将某个元素拖动到另一个目标元素上

`def drag_and_drop(self, origin_el, destination_el):`

* origin_el: 被拖拽的元素
* destination_el: 目标元素

## 文本输入

``` python
phone = wait.until(EC.presence_of_element_located((By.ID, 'com.tencent.mm:id/ht')))
phone.set_text('18888888888')
```

## 动作链

于 Selenium 中的 ActionChains 类似，Appium 中的 TouchAction 可支持的方法有 tap(), press(), long_press(), release(), move_to(), wait(), cancel() 等，示例如下

``` python
from appium.webdriver.common.touch_action import TouchAction

el = driver.find_element_by_id('el_id')
action = TouchAction(driver)
action.tap(el).perform()
```

实现拖动

``` python
els = driver.find_element_by_class_name('listview')
action1 = TouchAction(driver)
action1.press(els[0]).move_to(x=10, y=0).move_to(x=10, y=-75).release()
```