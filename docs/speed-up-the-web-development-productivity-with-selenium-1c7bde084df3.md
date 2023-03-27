# 使用 Selenium 加快 Web 开发效率

> 原文：<https://itnext.io/speed-up-the-web-development-productivity-with-selenium-1c7bde084df3?source=collection_archive---------3----------------------->

![](img/bc6cd05ae61b07df112b38ff3305c24e.png)

想象一下，你正在开发一个网站，每次重启，修复 bug 或者添加功能。你已经经历了一个重复的过程，比如用不同的用户登录，然后回答安全问题，然后点击几个标签，以便去你想要工作的地方。

每一步可能只需要几秒钟，但从长远来看，它会让你付出更多。我真的讨厌重复。太无聊了！

然后，有一个名为 Selenium 的工具凭空出现并保存日期——它不是凭空出现的。这是一套专门用于自动化网络浏览器的工具。

在这篇文章中，我将使用 [Selenium-Python](https://selenium-python.readthedocs.io/index.html) 来加速我的 web 开发过程。在继续下一步之前，请阅读他们的说明，将 Python 和 Selenium 安装到您的机器上。

```
OS: MacOS Sierra
Editor: Visual Studio Code
Project: AngularJS
Browser: Chrome (remember to install [chromedriver](https://sites.google.com/a/chromium.org/chromedriver/downloads))
```

简而言之，我会用 Python 写一小块代码，它会帮助我打开 Chrome 浏览器，用用户名和密码登录，然后回答我的安全密码，这样我就再也不用这么做了。这是最基本的。

**创建你的 python 测试文件**

```
// test.pyimport seleniumfrom selenium import webdriver# Using Chrome to open the web
browser = webdriver.Chrome()# Open the website
browser.get('http://your-website.com')
```

通过运行以下命令测试 python 文件:

```
// python3 if you are using version 3
python test.py
```

确保没有错误，它将在新的 Chrome 浏览器中打开[http://your-website.com](http://your-website.com)。

**尝试通过您的测试文件**登录

我注意到的一件事是，有很多网站不会自动加载，它必须等待几秒钟，你的元素才会出现在浏览器上。所以在您的测试文件中，最好在测试之前等待元素出现。

```
// test.py
import seleniumfrom selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By
from selenium.common.exceptions import TimeoutException# Using Chrome to open the web
browser = webdriver.Chrome()# Open the website
browser.get('http://your-website.com['](http://localhost:3000/#!/en/iol/landing'))**delay = 3 # seconds
try:
    WebDriverWait(browser, delay).until(EC.presence_of_element_located((By.ID, 'username')))
    print ("Username Input is ready!")
except TimeoutException:
    print ("Loading took too much time!")**
```

例如，上面的代码将加载页面，并等待 Id = username 的元素出现。我假设您知道如何获得每个元素的 id，要么通过您的代码，要么通过浏览器检查器。

现在，我们可以自动化我们的登录过程:

```
// test.py
"""
Start to login
"""# select the log Id box
id_box = browser.find_element_by_id('**username**')# Send id information
id_box.send_keys('dalenguyen')# Password enter
pass_box = browser.find_element_by_id('**password**')
pass_box.send_keys('123456')# Find login button
login_button = browser.find_element_by_id('**login-submit**')
login_button.click()
```

现在，如果你运行***python test . py***，它会打开一个新的浏览器，自动输入用户名+密码，然后让你登录。

我觉得你玩 [selenium-python](https://selenium-python.readthedocs.io/index.html) 就够了。该应用程序不仅限于登录你的帐户，但你可以上传文件，拖放，到不同的标签或行动，只要你能找到元素通过他们的 ID，类名，标签名，或 XPath…你应该阅读他们的文件为您的信息。

如果你对这个更友好的 UI 版本感兴趣，你可以试试 Chrome 和 Firefox 的 [Kantu 浏览器自动化](https://a9t9.com/kantu)插件。这是迄今为止我试过的最好的一个。

希望这将有助于改善您的测试或开发过程；)