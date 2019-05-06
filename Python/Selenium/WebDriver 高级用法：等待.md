# WebDriver 高级用法：等待

---

## 等待

---

在用 WebDriver 控制浏览器进行自动化操作时，有时候需要等待。根据个人经验需要等待的场景如下：

- 等待页面或某个元素加载完毕。比如请求一个页面后，我们需要等待一会，确保页面加载完毕，然后访问页面上的元素才不会出错。
- 避免访问过于频繁而被目标的反爬机制检测出来。有种比较简单的反爬机制就是限制用户在一段时间内的访问频率。比如账号在几秒钟内翻了十几个页面，那肯定就是爬虫。所以为了不被这种反爬机制检测出来，可以在每请求一个页面后随机等待一段时间。
- 避免鼠标移动太快而被网站的人机检测命中。这和前一条有点差别。有的网站会记录用户鼠标移动的轨迹，在前端或后端根据用户鼠标移动轨迹判断当前是真人还是爬虫程序在访问网站。鼠标轨迹判断有个简单的规则就是真人移动鼠标的速度不可能太快，而爬虫可以瞬移鼠标。所以这种情况下，我们需要慢慢移动鼠标，即每移动一下鼠标就等待一会。

### 显示等待

显示等待，等待某个确定的条件发生后才进行下一步操作。最粗暴的显示等待就是调用 `Thread.sleep()` 或 `time.sleep()` 等待确定的一段时间。在 Selenium 中组合使用 `WebDriverWait` 和 `ExpectedCondition` 可以很方便的等待某个条件发生。

例如等待一个元素出现在 DOM 树上，并设置 10 秒超时：

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

browser = webdriver.Firefox()
browser.get("http://somedomain/url_that_delays_loading")
try:
    element = WebDriverWait(ff, 10).until(EC.presence_of_element_located((By.ID, "myDynamicElement")))
finally:
    browser.quit()
```

`presence_of_element_located` 表示期待某个元素出现在 DOM 树上，并返回该 Web 元素。注意元素出现在 DOM 树上并不表示元素在页面上可见。如果想要等待一个元素在页面上可见，可以使用 `visibility_of_element_located`。下面的例子，等待 weibo.cn 页面的登陆按钮可见，并设置饿了 20 秒的等待超时。

```python
browser.get(url='https://weibo.cn/')
# 等待登陆按钮出现
wait = WebDriverWait(browser, 20)
login_a = wait.until(expected_conditions.visibility_of_element_located(
    (By.CSS_SELECTOR, 'a[href*="passport.weibo.cn/signin/login"]')))
```

`visibility_of_element_located` 等待元素在页面上可见并返回元素。

默认情况下，`WebDriverWait` 每过 500 毫秒调用一次 `ExpectedCondition` 直到它返回 true 或者一个非 null 对象，或者直到超时。

### 隐式等待

隐式等待，告诉 WebDriver 在查找元素时，如果元素不在，可以轮询一段时间。默认情况下 WebDriver 轮询等待的时间为 0，即不轮询。

```python
from selenium import webdriver

ff = webdriver.Firefox()
ff.implicitly_wait(10) # seconds
ff.get("http://somedomain/url_that_delays_loading")
myDynamicElement = ff.find_element_by_id("myDynamicElement")
```

 上面的例子中，将 WebDriver 查找元素时轮询等待的时间上限设为 10 秒。后续如果查找一个暂时不存在的元素时会最多轮询10秒。







