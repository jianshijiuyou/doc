# 基本用法

``` python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.wait import WebDriverWait

browser = webdriver.Chrome()

try:
    browser.get('https://www.baidu.com')
    input_ = browser.find_element_by_id('kw')
    input_.send_keys('Python')
    input_.send_keys(Keys.ENTER)
    wait = WebDriverWait(browser, 10)
    wait.until(EC.presence_of_element_located((By.ID, 'content_left')))
    print(browser.current_url)
    print(browser.get_cookies())
    print(browser.page_source)
finally:
    browser.close()
```

# 声明浏览器对象

``` python
from selenium import webdriver

browser = webdriver.Chrome()
browser = webdriver.Firefox()
browser = webdriver.Edge()
browser = webdriver.PhantomJS()
browser = webdriver.Safari()
```

# 访问页面

``` python
from selenium import webdriver

browser = webdriver.Chrome()
browser.get('https://www.taobao.com')
print(browser.page_source)
browser.close()
```

# 查找节点

## 单个节点

``` python
from selenium import webdriver

browser = webdriver.Chrome()
browser.get('https://www.taobao.com')
input1 = browser.find_element_by_id('q')
input2 = browser.find_element_by_css_selector('#q')
input3 = browser.find_element_by_xpath('//*[@id="q"]')
print(input1)
print(input2)
print(input3)
browser.close()
```

所有查出找方法

``` python
find_element_by_id
find_element_by_name
find_element_by_xpath
find_element_by_link_text
find_element_by_partial_link_text
find_element_by_tag_name
find_element_by_class_name
find_element_by_css_selector
```

## 多个节点

多个节点查找只需多加上 `s` 就行了，返回列表

``` python
browser.get('https://www.taobao.com')
# lis = browser.find_elements_by_css_selector('.service li')
lis = browser.find_elements(By.CSS_SELECTOR ,'.service li')
```

其他方法一样

# 节点交互

`send_keys`: 输入文字

`clear`: 清空文字

`click`: 点击按钮


``` python
from selenium import webdriver
import time

browser = webdriver.Chrome()
browser.get('https://www.taobao.com')
input_ = browser.find_element_by_id('q')
input_.send_keys('iPhone')
time.sleep(1)
input_.clear()
input_.send_keys('iPad')
button = browser.find_element_by_class_name('btn-search')
button.click()
```

# 动作链

``` python
from selenium import webdriver
from selenium.webdriver import ActionChains

browser = webdriver.Chrome()
url = 'http://www.runoob.com/try/try.php?filename=jqueryui-api-droppable'
browser.get(url)
browser.switch_to_frame('iframeResult') # 切换 frame
source = browser.find_element_by_css_selector('#draggable')
target = browser.find_element_by_css_selector('#droppable')
actions = ActionChains(browser)
actions.drag_and_drop(source, target)
actions.perform() # 开始拖动
```

# 执行 JavaScript

自动下拉进度条

``` python
browser = webdriver.Chrome()
url = 'https://www.zhihu.com/explore'
browser.get(url)
browser.execute_script('window.scrollTo(0, document.body.scrollHeight)')
browser.execute_script('alert("To Bottom")')
```

# 获取节点信息

## 获取属性

get_attribute() 方法用于获取节点属性

``` python
browser = webdriver.Chrome()
url = 'https://www.zhihu.com/explore'
browser.get(url)
logo = browser.find_element_by_id('zh-top-link-logo')
print(logo)
print(logo.get_attribute('class'))
```

## 获取文本值

text 属性

``` python
url = 'https://www.zhihu.com/explore'
browser.get(url)
question = browser.find_element_by_class_name('zu-top-add-question')
print(question.text)
```

## id、位置、标签名、大小

``` python
question = browser.find_element_by_class_name('zu-top-add-question')
print(question.id)
print(question.location)
print(question.tag_name)
print(question.size)

# 0.041960035468208146-1
# {'x': 762, 'y': 7}
# button
# {'height': 32, 'width': 66}
```

# 切换 Frame

selenium 默认是再父级 Frame 中操作，此时获取不到子 Frame 里面的节点，需要使用 switch_to_frame() 方法切换 Frame。

``` python
browser.switch_to_frame('iframeResult')
browser.switch_to.parent_frame() # 切换回父级 Frame
```

# 延时等待

get() 方法会再网页框架加载结束后结束执行，但是如果有 Ajax 请求，此时用 page_source 获取源码可能不一定是我们想要的。需要延时等待一段时间。

## 隐式等待

超出指定时间还是找不到节点抛出异常

implicitly_wait()

``` python
browser = webdriver.Chrome()
url = 'https://www.zhihu.com/explore'
browser.implicitly_wait(10)
browser.get(url)
question = browser.find_element_by_class_name('zu-top-add-question')
```

## 显式等待

指定时间内找到节点则提前结束等待，否则抛出异常

``` python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.wait import WebDriverWait

browser = webdriver.Chrome()
browser.get('https://www.taobao.com')
wait = WebDriverWait(browser, 10)
input_q = wait.until(EC.presence_of_element_located((By.ID, 'q')))
button = wait.until(EC.element_to_be_clickable((By.CSS_SELECTOR, '.btn-search')))
print(input_q)
print(button)

```

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/images/20180628170614.png)

# 前进后退

``` python
browser.back() # 后退
browser.forward() # 前进
```

# Cookies

``` python
browser.get_cookies()
browser.add_cookie({'name': 'name', 'domain': 'www.zhihu.com'})
browser.delete_all_cookies()
```

# 选项卡管理

``` python
from selenium import webdriver
import time

browser = webdriver.Chrome()
browser.get('https://www.taobao.com')
browser.execute_script('window.open()')
print(browser.window_handles)
browser.switch_to_window(browser.window_handles[1])
browser.get('https://www.baidu.com')
time.sleep(1)
browser.switch_to_window(browser.window_handles[0])
browser.get('https://python.org')
```

# 异常处理

``` python
from selenium import webdriver
from selenium.common.exceptions import TimeoutException, NoSuchElementException


browser = webdriver.Chrome()
try:
    browser.get('https://www.baidu.com')
except TimeoutException:
    print('time out')
try:
    hello = browser.find_element_by_id('hello')
except NoSuchElementException:
    print('no element')
finally:
    browser.close()
```

# 代理设置

chrome

``` python
from selenium import webdriver

proxy = '127.0.0.1:1080'
chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument('--proxy-server=http://' + proxy)
browser = webdriver.Chrome(chrome_options=chrome_options)
browser.get('http://httpbin.org/get')
```

PhantomJS

``` python
from selenium import webdriver

service_args = [
    '--proxy=127.0.0.1:1080',
    '--proxy-type=http',
    # '--proxy-auth=username:passwrod'
]
browser = webdriver.PhantomJS(service_args=service_args)
browser.get('http://httpbin.org/get')

```

# Chrome Headless 模式

使用 Chrome 的无界面模式（59 版本以上）

``` python
chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument('--headless')
browser = webdriver.Chrome(chrome_options=chrome_options)
```

# PhantomJS 缓存配置

禁用图片加载，配置缓存

``` python
SERVICE_AGES = ['--load-images=false', '--disk-cache=true']
browser = webdriver.PhantomJS(service_args=SERVICE_AGES)
```

> 最新的 Selenium 已经不推荐使用 PhantomJS 了，请使用 Chrome 的 headless 模式

# 淘宝数据抓取

``` python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.wait import WebDriverWait
from selenium.common.exceptions import TimeoutException
from urllib.parse import quote
from pyquery import PyQuery as pq
from pymongo import MongoClient


chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument('--headless')
browser = webdriver.Chrome(chrome_options=chrome_options)
wait = WebDriverWait(browser, 10)

client = MongoClient()
taobao = client.taobao
ipad = taobao.ipad

KEYWORD = 'iPad'

def get_page(page):
    try:
        print('正在抓取第 {} 页'.format(page))
        browser.get('https://s.taobao.com/search?q={}'.format(KEYWORD))
        if page > 1:
            input_ = wait.until(EC.presence_of_element_located((By.CSS_SELECTOR, '#mainsrp-pager div.form .input')))
            button = wait.until(EC.element_to_be_clickable((By.CSS_SELECTOR, '#mainsrp-pager div.form .btn')))
            input_.clear()
            input_.send_keys(page)
            button.click()
            ############## 页面跳转完成
        # 等待页面加载
        wait.until(EC.text_to_be_present_in_element((By.CSS_SELECTOR, '#mainsrp-pager .item.active > span'), str(page)))
        wait.until(EC.presence_of_element_located((By.CSS_SELECTOR, '#mainsrp-itemlist .items .item')))
        get_products()
    except TimeoutException:
        get_page(page)

def get_products():
    html = browser.page_source
    doc = pq(html.replace('xmlns', 'another_attr'))
    items = doc('#mainsrp-itemlist .items .item').items()
    for item in items:
        product = {
            'img': item.find('.J_ItemPic.img').attr('data-src'),
            'title': item.find('.title').text(),
            'price': item.find('.price strong').text(),
            'deal': item.find('.deal-cnt').text(),
            'shopname': item.find('.shopname').text(),
            'location': item.find('.location').text()
        }
        # print(product)
        save_to_mongo(product)

def save_to_mongo(product):
    ipad.insert_one(product)

if __name__ == '__main__':
    for page in range(10, 20):
        get_page(page)
```