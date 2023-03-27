# 如何在 Selenium WebDriver 中运行无头 Chrome 浏览器？

> 原文：<https://itnext.io/how-to-run-a-headless-chrome-browser-in-selenium-webdriver-c5521bc12bf0?source=collection_archive---------0----------------------->

![](img/929a550e3a78f0ff487cc6d3ccf7f8fc.png)

照片由[沃特·贝杰特](https://unsplash.com/photos/yvEkwjSBZIY)拍摄

从版本 60 开始，Chrome 浏览器引入了在 T4 无头模式下运行的能力。我们现在能够在不创建可视浏览器窗口的情况下启动浏览器。这将允许您用更少的资源更快地运行测试，最重要的是，它将允许您在没有图形组件的系统上运行测试。不要担心，截屏功能不会受到任何影响。

在本文中，我想演示如何使用这个功能。

任何自动测试的开始都是标准的，我们指定 chrome 驱动程序的路径:

```
System.setProperty(**"webdriver.chrome.driver"**, **"/path/to/driver"**);
```

这件事情之后，我们要做的就是创建一个 WebDriver 对象，并设置 ChromeDriver 路径和一些参数:

```
ChromeOptions options = new ChromeOptions();options.addArguments("--headless");WebDriver driver = new ChromeDriver(options);
```

仅此而已。你可以运行它。

但是，有一个问题。不可见的浏览器窗口只有 800x600 大小。因此，您需要使用一个附加参数来设置所需的屏幕大小:

```
ChromeOptions options = new ChromeOptions();options.addArguments("--headless", "--window-size=1920,1200");WebDriver driver = new ChromeDriver(options);
```

通常，您还应该指定在开发人员模式下禁用 GPU 渲染、扩展和弹出窗口扩展。这通常会导致非常难看和不可读的代码。

```
ChromeOptions options = new ChromeOptions();options.addArguments("--headless", "--disable-gpu", "--window-size=1920,1200","--ignore-certificate-errors","--disable-extensions","--no-sandbox","--disable-dev-shm-usage");WebDriver driver = new ChromeDriver(options);
```

为了获得更具可读性的视图，我们将重构代码:

*   让我们从指定驱动程序的路径开始。我们可以使用第三方的 [WebDriverManager](https://github.com/bonigarcia/webdrivermanager) 库。

```
System.setProperty(**"webdriver.chrome.driver"**, **"/path/to/driver"**);
```

![](img/8fbcf60db3f883b1f89736dcac22f012.png)

```
@BeforeClass
    **public static void** setupClass() {
        WebDriverManager.*chromedriver*().setup();
    }
```

*   你需要创建一个实例 Chrome 浏览器。

```
ChromeOptions options = new ChromeOptions();options.addArguments("--headless", "--disable-gpu", "--window-size=1920,1200","--ignore-certificate-errors","--disable-extensions","--no-sandbox","--disable-dev-shm-usage");WebDriver driver = new ChromeDriver(options);
```

![](img/8fbcf60db3f883b1f89736dcac22f012.png)

```
*/**
 * The class Chrome. This class describes Chrome browser instance.
 */* **public class** Chrome **implements** WebDriverProvider {
    */**
     * The constant LOG.
     */* **private static final** Logger ***LOG*** = Logger.*getLogger*(Chrome.**class**.getName());

    */**
     * The method createDriver.
     */* @Override
    **public** WebDriver createDriver(**final** DesiredCapabilities capabilities) {
        WebDriverManager.*chromedriver*().setup();
        capabilities.setCapability(ChromeOptions.***CAPABILITY***, *getChromeOptions*());
        **try** {
            **return new** ChromeDriver(*getChromeOptions*());
        } **catch** (Exception ex) {
            **if** (***LOG***.isLoggable(Level.***INFO***)) {
                ***LOG***.info(String.*valueOf*(ex));
            }
        }
        **return null**;
    }

    */**
     * Method getChromeOptions.
     *
     ** ***@return*** *the chrome options.
     */* **public static** ChromeOptions getChromeOptions() {
        **final** ChromeOptions chromeOptions = **new** ChromeOptions();
        chromeOptions.addArguments(**"disable-infobars"**);
        chromeOptions.setExperimentalOption(**"excludeSwitches"**, Collections.*singletonList*(**"enable-automation"**));
        chromeOptions.setExperimentalOption(**"useAutomationExtension"**, **false**);

        chromeOptions.addArguments(**"--disable-gpu"**);
        chromeOptions.addArguments(**"--disable-extensions"**);
        chromeOptions.addArguments(**"--no-sandbox"**);
        chromeOptions.addArguments(**"--disable-dev-shm-usage"**);
        chromeOptions.addArguments(**"--headless"**);
        chromeOptions.addArguments(**"--window-size=1580,1280"**);

        **final** HashMap<String, Object> prefs = **new** HashMap();
        prefs.put(**"credentials_enable_service"**, **false**);
        prefs.put(**"profile.password_manager_enabled"**, **false**);
        chromeOptions.setExperimentalOption(**"prefs"**, prefs);

        **return** chromeOptions;
    }
}
```

我们需要创建一个接口 WebDriverProvider:

```
*/**
 * The interface Web driver provider.
 */* **public interface** WebDriverProvider {

    */**
     * Create new {****@link*** *WebDriver} instance.
     *
     ** ***@param capabilities*** *set of desired capabilities
     *                     as suggested by Selenium framework;
     ** ***@return*** *new {****@link*** *WebDriver} instance.
     */* WebDriver createDriver(DesiredCapabilities capabilities);

}
```

经过这次重构，代码变得更具可读性和可重用性。例如，您可以用同样的方式为 Firefox 浏览器创建一个实例:

```
*/**
 * The class Firefox.
 */* **public class** Firefox **implements** WebDriverProvider {

    */**
     * The method for createDriver for Firefox.
     */* @Override
    **public** WebDriver createDriver(**final** DesiredCapabilities capabilities) {
        WebDriverManager.firefoxdriver().setup();

        **return new** FirefoxDriver(*getFirefoxOptions*().merge(capabilities));
    }

    */**
     * Method getFirefoxOptions.
     *
     ** ***@return*** *the firefox options
     */* **public static** FirefoxOptions getFirefoxOptions() {
        **final** FirefoxOptions firefoxOptions = **new** FirefoxOptions();
        firefoxOptions.addArguments(**"--disable-web-security"**);
        firefoxOptions.addArguments(**"--allow-running-insecure-content"**);

        **final** FirefoxProfile profile = **new** FirefoxProfile();
        profile.setAcceptUntrustedCertificates(**true**);
        profile.setAssumeUntrustedCertificateIssuer(**false**);
        profile.setPreference(**"pageLoadStrategy"**, **"normal"**);

        firefoxOptions.setCapability(FirefoxDriver.PROFILE, profile);

        **return** firefoxOptions;
    }
}
```

在基类中，调用:

```
**final** DesiredCapabilities capabilities = **new** DesiredCapabilities().*chrome*();
**final** Chrome chrome = **new** Chrome();DriverHolder.*setDriverThread*(chrome.createDriver(capabilities));
```

并在 DriverHolder 中创建一个对驱动线程的调用:

```
*/**
 * The class Driver holder.
 */* **public final class** DriverHolder {

    */**
     * The value DRIVER_THREAD.
     */* **private static final** ThreadLocal<WebDriver> ***DRIVER_THREAD*** = **new** ThreadLocal<>();

    */**
     * The constructor.
     */* **private** DriverHolder() {
    }

    */**
     * Gets thread driver.
     *
     ** ***@return*** *the thread driver
     */* **public static** WebDriver getDriverThread() {
        **return *DRIVER_THREAD***.get();
    }

    */**
     * Sets thread driver.
     *
     ** ***@param webDriverValue*** *the web driver value
     */* **public static void** setDriverThread(**final** WebDriver webDriverValue) {
        ***DRIVER_THREAD***.set(webDriverValue);
    }
}
```

正如你所看到的，Chrome headless 真的很容易使用，它与 PhantomJS 没有太大的不同，因为我们使用 Selenium 来运行它。