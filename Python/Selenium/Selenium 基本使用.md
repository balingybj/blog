## 创建一个浏览器驱动实例

```python
from selenium import webdriver
# Chrome 浏览器
browser = webdriver.Chrome()
# 火狐 浏览器
browser = webdriver.Firefox()
```

## 请求一个页面

```python
browser.get("http://www.baidu.com")
```

`browser.get()` 可能在页面请求开始、加载中、或加载完任意一个阶段返回。具体在哪个阶级返回取决于操作系统和浏览器的版本组合。所以最好通过等待页面上的具体元素是判断页面是否加载完毕。

## 查看当前页面标题

```python
print(browser.title)
```

## 定位  Web 元素

浏览器驱动提供了查找单个元素和多个元素的方法，查找到的元素也提供了查找单个或多个子元素的方法。如果查找元素失败则抛出异常。

一般方法 `find_element` 或 `find_element_xxx` 表示查找单个元素， `find_elements` 或 `find_elements_xxx` 表示查找多个元素。

### 通过  id  查找元素

比如查找 id 为 "loginName" 的输入框。

```python
element = browser.find_element_by_id("loginName")
# 或者
from selenium.webdriver.common.by import By
element = browser.find_element(by=By.ID, value="loginName")
```

### 通过 class 名称

查找所有 class 为 apple 的元素。

```pyt
apples = driver.find_elements_by_class_name("apple")
# 或者
from selenium.webdriver.common.by import By
apples = driver.find_elements(By.CLASS_NAME, "apple")
```

### 通过标签名

比如查找所有标签名为 table 的元素。

```python
table = driver.find_element_by_tag_name("table")
# 或者
from selenium.webdriver.common.by import By
table = driver.find_element(By.TAG_NAME, "table")
```

### 通过 name 属性查找输入元素

假设有个页面元素的代码如下：

```html
<input type="tel" name="phoneNo" class="Input" placeholder="手机号">
```

则可以通过其 name 属性值定位到它：

```python
phone = driver.find_element_by_name("phoneNo")
# 或者
from selenium.webdriver.common.by import By
phone = driver.find_element(By.NAME, "phoneNo")
```

### 通过链接文本

有的文本背后包含一个链接，点击后会触发新的网络请求。比如 weibo.cn 的未登陆页面有一个登陆链接如下：

```html
<a href="https://passport.weibo.cn/signin/login?entry=mweibo&amp;r=https%3A%2F%2Fweibo.cn%2F&amp;backTitle=%CE%A2%B2%A9&amp;vt=">登录</a>
```

可以通过文本“登陆”来查找该元素：

```python
login = driver.find_element_by_link_text("登陆")
# 或者
from selenium.webdriver.common.by import By
login = driver.find_element(By.LINK_TEXT, "登陆")
```

### 通过部分链接文本

假如链接文本太长，或者有多个链接文本，这些链接文本有部分文字相同，则可以通过部分文本来查找它们。

假如有几个 a 元素如下：

```html
<a href="http://www.google.com/search">谷歌搜索</a>
<a href="http://www.baidu.com">百度搜索</a>
```

可以通过文本“搜索”来查找它们：

```python
search = driver.find_element_by_partial_link_text("搜索")
# 或者
from selenium.webdriver.common.by import By
search = driver.find_element(By.PARTIAL_LINK_TEXT, "搜索")
```

### 通过 CSS 选择器

通过 CSS 选择器可以完成比较复杂的元素定位。

查找 div 元素的子元素中 href 属性包含 “passport.weibo.cn/signin/login” 的 a 元素：

```python
login = driver.find_element_by_css_selector('div > a[href*="passport.weibo.cn/signin/login"]')
# 或者
login = driver.find_element(By.CSS_SELECTOR, 'div > a[href*="passport.weibo.cn/signin/login"]')
```

### 通过  XPath 表达式

浏览器驱动可以利用浏览器原生的 XPath 功能。对于没有原生的 XPath 支持的浏览器，Selenium 还提供了额外的 XPath 支持。但是不同的浏览器原生的 XPath 和浏览器驱动提供的 XPath 支持可能有细微的差异。

| 驱动                                                         | 标签和属性名 | 属性值       | 是否有元素的 XPath 支持 |
| ------------------------------------------------------------ | ------------ | ------------ | ----------------------- |
| [HtmlUnit Driver](https://www.seleniumhq.org/docs/03_webdriver.jsp#htmlunit-driver) | 小写         | 同 HTML 代码 | 是                      |
| [Internet Explorer Driver](https://www.seleniumhq.org/docs/03_webdriver.jsp#internet-explorer-driver) | 小写         | 同 HTML 代码 | 否                      |
| [Firefox Driver](https://www.seleniumhq.org/docs/03_webdriver.jsp#firefox-driver) | 不区分大小写 | 同 HTML 代码 | 是                      |

例子，假设有如下 HTML：

```html
<input type="text" name="example" />
<INPUT type="text" name="other" />
```

对于不同的浏览器驱动，执行如下同一份代码：

```python
inputs = driver.find_elements_by_xpath("//input")
# 或者
from selenium.webdriver.common.by import By
inputs = driver.find_elements(By.XPATH, "//input")
```

各个浏览器驱动匹配到的结果如下：

| XPath 表达式 | [HtmlUnit Driver](https://www.seleniumhq.org/docs/03_webdriver.jsp#htmlunit-driver) | [Firefox Driver](https://www.seleniumhq.org/docs/03_webdriver.jsp#firefox-driver) | [Internet Explorer Driver](https://www.seleniumhq.org/docs/03_webdriver.jsp#internet-explorer-driver) |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| //input      | 1                                                            | 2                                                            | 2                                                            |
| //INPUT      | 0                                                            | 2                                                            | 0                                                            |

### 通过 JavaScript

还可以通过执行一段 JavaScript 代码来查找元素，只要该 JavaScript 代码返回 DOM 元素，浏览器驱动会自动将 JavaScript 代码返回的 DOM 元素转化成 Web 元素。

查找第一个 class=“chesse” 的元素

```python
element = driver.execute_script("return $('.cheese')[0]")
```

查找所有 label 元素的 input 子元素：

```python
abels = driver.find_elements_by_tag_name("label")
inputs = driver.execute_script(
    "var labels = arguments[0], inputs = []; for (var i=0; i < labels.length; i++){" +
    "inputs.push(document.getElementById(labels[i].getAttribute('for'))); } return inputs;", labels)
```

## 获取文本值

```python
element = driver.find_element_by_id("element_id")
element.text
```

`element.text` 只会返回在 web 页面上可见的文本值。

## 填写表单

等到有具体例子再写

## 在窗口和 Frames 之间移动

。。。

## 弹窗

。。。

## Cookies

一般浏览器会自动管理 cookie，但如果你想在请求某个页面之前提前设置好 cookie，或者获取当前页面的 cookie，浏览器驱动也提供了方法：

```python
# 请求一个页面
driver.get("http://www.example.com")

# 设置 cookie，这里的的 cookie 名为 'key'，cookie 值为 'value'
driver.add_cookie({'name':'key', 'value':'value', 'path':'/'})
# 其他可选参数包括:
# 'domain' -> String,
# 'secure' -> Boolean,
# 'expiry' -> 过期时间，毫秒。Milliseconds since the Epoch it should expire.

# 获取当前页面所有 cookie
for cookie in driver.get_cookies():
    print(cookie['name'], cookie['value'])

# 删除指定 cookie
driver.delete_cookie("CookieName")
# 删除所有 cookie
driver.delete_all_cookies()
```

我们可以使用 Selenium 去掉浏览器模拟登陆某个网站，获取到 cookie 后给爬虫程序使用。

## 设置 User Agent

火狐浏览器

```python
profile = webdriver.FirefoxProfile()
profile.set_preference("general.useragent.override", "some UA string")
driver = webdriver.Firefox(profile)
```

## 拖放

拖放即模拟用户在某个 web 元素上按下鼠标左键，将该元素拖动到另一个元素上，然后释放鼠标左键。

> 拖放需要 native event 被允许。
>
> todo:弄清楚 selenium 中的 event 是什么

```python
from selenium.webdriver.common.action_chains import ActionChains
element = driver.find_element_by_name("source")
target =  driver.find_element_by_name("target")
ActionChains(driver).drag_and_drop(element, target).perform()
```

