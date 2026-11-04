# 使用 Playwright 绕过 CAPTCHA

[![推广](https://github.com/bright-cn/Awesome-Web-Scraping/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner%20CN.png)](https://www.bright.cn/)

本指南将介绍如何使用 Playwright 绕过 CAPTCHA，确保你的网页爬虫任务顺畅运行而不中断：

- [什么是 CAPTCHA？可以绕过吗？](#什么是-captcha可以绕过吗)
- [使用 Playwright 绕过 CAPTCHA：分步教程](#使用-playwright-绕过-captcha分步教程)
  - [第 1 步：初始化 Node.js 项目](#第-1-步初始化-nodejs-项目)
  - [第 2 步：安装 Playwright Extra](#第-2-步安装-playwright-extra)
  - [第 3 步：设置 Playwright 脚本](#第-3-步设置-playwright-脚本)
  - [第 4 步：实现浏览器自动化逻辑](#第-4-步实现浏览器自动化逻辑)
  - [第 5 步：安装 Playwright Stealth 插件](#第-5-步安装-playwright-stealth-插件)
  - [第 6 步：注册隐身设置](#第-6-步注册隐身设置)
  - [第 7 步：再次进行机器人检测测试](#第-7-步再次进行机器人检测测试)
- [如果 Playwright CAPTCHA 破解方案不起作用怎么办？](#如果-playwright-captcha-破解方案不起作用怎么办)
 
## 什么是 CAPTCHA？可以绕过吗？

[CAPTCHA](https://www.bright.cn/blog/web-data/what-is-a-captcha) 是“用于区分计算机和人类的全自动公开图灵测试”（Completely Automated Public Turing tests to tell Computers and Humans Apart）的缩写，是一种用于区分人类用户与自动化机器人的测试。人类通常可以轻松完成，但机器在设计上应难以通过。

![CAPTCHA 示例](https://github.com/bright-cn/bypass-captcha-with-playwright/blob/main/images/Example-of-a-CAPTCHA.png)

Google reCAPTCHA、hCaptcha 和 BotDetect 是一些最受欢迎的 CAPTCHA 提供商。它们通常支持以下一种或多种 CAPTCHA 类型：

- <strong>图像验证：</strong>用户必须从一组网格图片中识别特定对象。
- <strong>文本验证：</strong>用户需要输入一串经过扭曲处理的字母和数字。
- <strong>音频验证：</strong>用户需要输入听到的单词。
- <strong>拼图验证：</strong>用户必须完成一个简单的拼图，例如将拼图块滑动到正确位置。

CAPTCHA 可以作为特定用户流程的一部分，例如提交表单的最后一步：

![将 CAPTCHA 作为表单提交步骤的示例](https://github.com/bright-cn/bypass-captcha-with-playwright/blob/main/images/CAPTCHA-as-a-step-of-a-form-submission-process-example.png)

在这些情况下，CAPTCHA 始终会显示，机器人无法避开。不过，你可以将软件与 CAPTCHA 破解库集成以实现自动处理，或使用依靠人工操作员实时解决这些验证挑战的服务。

CAPTCHA 也常被用作更广泛的反机器人解决方案的一部分，例如[网页应用防火墙](https://www.cloudflare.com/learning/ddos/glossary/web-application-firewall-waf/)：

![网页应用防火墙示例](https://github.com/bright-cn/bypass-captcha-with-playwright/blob/main/images/Example-of-a-Web-Application-Firewall-1024x488.png)

当这些系统怀疑用户可能是机器人时，会动态显示 CAPTCHA。在这种情况下，可以让机器人模仿人类行为来绕过 CAPTCHA。

## 使用 Playwright 绕过 CAPTCHA：分步教程

避开 CAPTCHA 的一种高效方法，是在使用类人指纹的同时，通过自动化脚本模拟人类行为。[Playwright](https://playwright.dev/) 是领先的浏览器自动化库，也是实现此目的的最佳工具之一。

本指南的教程部分将介绍如何实现使用 Playwright 绕过 CAPTCHA 的逻辑。

### 第 1 步：初始化 Node.js 项目

如果你已经有用于网页爬虫或测试的 Playwright 脚本，请跳过此步骤。否则，请为 Playwright CAPTCHA 破解项目创建一个文件夹，然后在终端中进入该文件夹：

```bash
mkdir playwright_demo

cd playwright_demo
```

在其中初始化一个新的 Node.js 项目：

```bash
npm init -y
```

在你常用的 JavaScript IDE 中打开项目文件夹，并添加一个新的 `script.js` 文件。

![在 IDE 中添加新的 script.js 文件](https://github.com/bright-cn/bypass-captcha-with-playwright/blob/main/images/Adding-a-new-script.js-file-in-the-IDE.png)

不要忘记打开 `package.json`，添加 `"type": "module"` 以[将项目标记为模块](https://nodejs.org/api/packages.html#type)。

### 第 2 步：安装 Playwright Extra

Playwright 本身不支持插件，但社区开发的 [Playwright Extra](https://github.com/berstend/puppeteer-extra/tree/master/packages/playwright-extra#readme) 弥补了这一不足。

将 `playwright` 和 [`playwright-extra`](https://www.npmjs.com/package/playwright-extra) 添加到项目依赖项中：

```bash
npm i playwright playwright-extra
```

### 第 3 步：设置 Playwright 脚本

现在需要初始化脚本，让 Playwright 破解 CAPTCHA 验证。要从 `playwright-extra` 导入你希望控制的浏览器，请将以下代码行添加到 `script.js`：

```js
import { chromium } from "playwright-extra"
```

初始化一个新的 [async](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) 函数，通过 Playwright API 执行类人交互：

```js
(async () => {

    // set up the browser and launch it

    const browser = await chromium.launch()

    // open a new blank page

    const page = await browser.newPage()

    // browser automation logic...

    // close the browser and release its resources

    await browser.close()

})()
```

这会启动一个新的 Chromium 实例并打开新页面，然后关闭浏览器。

### 第 4 步：实现浏览器自动化逻辑

目标网站是 [bot.sannysoft.com](https://bot.sannysoft.com/)，这是一个特殊网页，会在浏览器中运行一系列测试，以判断用户是人类还是机器人。如果你尝试在本地浏览器中访问此页面，应该会看到所有测试均已通过。

使用 [`goto()`](https://playwright.dev/docs/api/class-page#page-goto) 方法连接到目标页面：

```js
await page.goto("https://bot.sannysoft.com/")
```

现在为整个页面创建截图，以查看反机器人测试结果：

```js
await page.screenshot("results.png")
```

将所有代码整合在一起，即可得到以下 `script.js` 文件：

```js
import { chromium } from "playwright-extra"

(async () => {

    // set up the browser and launch it

    const browser = await chromium.launch()

    // open a new blank page

    const page = await browser.newPage()

    // navigate to the target page

    await page.goto("https://bot.sannysoft.com/")

    // take a screenshot of the entire page

    await page.screenshot({

        path: "results.png",

        fullPage: true

    })

    // close the browser and release its resources

    await browser.close()

})()
```

使用以下命令执行上述代码：

```bash
node script.js
```

该脚本会以无头模式打开 Chromium 实例、访问目标页面、截取屏幕截图，然后关闭浏览器。脚本执行结束后，项目根文件夹中会出现 `results.png` 文件，打开后将看到：

![results.png 文件示例](https://github.com/bright-cn/bypass-captcha-with-playwright/blob/main/images/results.png-file-example-196x1024.png)

原生 Playwright 在无头模式下无法通过多项测试。要解决此问题，请使用 Stealth 插件。

### 第 5 步：安装 Playwright Stealth 插件

[Playwright Stealth](https://github.com/berstend/puppeteer-extra/tree/39248f1f5deeb21b1e7eb6ae07b8ef73f1231ab9/packages/puppeteer-extra-plugin-stealth) 是用于 `playwright-extra` 的插件，可以防止机器人检测。它会覆盖多项配置，使浏览器实例看起来处于自然运行状态，就像未受 Playwright 控制一样。

Stealth 插件最初为 Puppeteer Extra 开发，但也适用于 Playwright Extra。使用 npm 安装：

```bash
npm i puppeteer-extra-plugin-stealth
```

接下来，使用以下代码行在 `script.js` 文件中导入 Stealth 插件：

```js
import StealthPlugin from "puppeteer-extra-plugin-stealth"
```

### 第 6 步：注册隐身设置

要实现 Playwright 绕过 CAPTCHA 的逻辑，只需通过 `use()` 方法在 `playwright-extra` 中注册 Stealth 插件：

```js
chromium.use(StealthPlugin())
```

现在，Playwright 控制的浏览器会表现得像由真实用户操作的普通浏览器。

### 第 7 步：再次进行机器人检测测试

目前，你的 `script.js` 文件应如下所示：

```js
import { chromium } from "playwright-extra"

import StealthPlugin from "puppeteer-extra-plugin-stealth"

(async () => {

    // register the Stealth plugin

    chromium.use(StealthPlugin())

    // set up the browser and launch it

    const browser = await chromium.launch()

    // open a new blank page

    const page = await browser.newPage()

    // navigate to the target page

    await page.goto("https://bot.sannysoft.com/")

    // take a screenshot of the entire page

    await page.screenshot({

        path: "results.png",

        fullPage: true

    })

    // close the browser and release its resources

    await browser.close()

})()
```

再次启动脚本：

```bash
node script.js
```

再次打开 `results.png`，现在你会看到所有机器人检测测试均已通过：

![results.png 文件的第二个示例：已通过机器人检测测试](https://github.com/bright-cn/bypass-captcha-with-playwright/blob/main/images/results.png-file-second-example-bot-detection-tests-passed-229x1024.png)

## 如果 Playwright CAPTCHA 破解方案不起作用怎么办？

浏览器设置并不是反机器人工具关注的唯一方面。IP 信誉也是一个关键因素，而免费库无法解决这一问题。

对于只需单击一次的简单 CAPTCHA，你可以使用 [`puppeteer-extra-plugin-recaptcha`](https://github.com/berstend/puppeteer-extra/tree/39248f1f5deeb21b1e7eb6ae07b8ef73f1231ab9/packages/puppeteer-extra-plugin-recaptcha) 插件。但在处理 Cloudflare 等更复杂的工具时，你需要更强大的解决方案。

如果你正在寻找真正的 Playwright CAPTCHA 破解工具，不妨尝试 Bright Data 的[网页爬虫解决方案](https://www.bright.cn/products/web-unlocker)。这些方案提供卓越的解锁能力，并配备专用的 [CAPTCHA 破解功能](https://www.bright.cn/products/web-unlocker/captcha-solver)，可自动处理 reCAPTCHA、hCaptcha、Cloudflare Turnstile、AWS WAF Captcha 及其他多种验证机制。

立即注册，今天就开始免费试用。
