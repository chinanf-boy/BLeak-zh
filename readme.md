# plasma-umass/BLeak [![explain]][source] [![translate-svg]][translate-list] 
    
<!-- [![size-img]][size] -->

[explain]: http://llever.com/explain.svg
[source]: https://github.com/chinanf-boy/Source-Explain
[translate-svg]: http://llever.com/translate.svg
[translate-list]: https://github.com/chinanf-boy/chinese-translate-list
[size-img]: https://packagephobia.now.sh/badge?p=Name
[size]: https://packagephobia.now.sh/result?p=Name
    


「 desc 」

[中文](./readme.md) | [english](https://github.com/plasma-umass/BLeak) 


---

## 校对 🀄️

<!-- doc-templite START generated -->
<!-- repo = 'plasma-umass/BLeak' -->
<!-- commit = 'f9d3c14722e88edcda6520e57533d519b3cce405' -->
<!-- time = '2018-11-20' -->

<!-- doc-templite END generated -->


### 贡献

欢迎 👏 勘误/校对/更新贡献 😊 [具体贡献请看](https://github.com/chinanf-boy/chinese-translate-list#贡献)
        

## 生活

[help me live , live need money 💰](https://github.com/chinanf-boy/live-need-money)

---

### 目录

<!-- START doctoc -->
<!-- END doctoc -->

# BLeak v1.2.1

[![Build Status](https://travis-ci.org/plasma-umass/BLeak.svg?branch=master)](https://travis-ci.org/plasma-umass/BLeak)
[![Build status](https://ci.appveyor.com/api/projects/status/b92sknh0pu38943q/branch/master?svg=true)](https://ci.appveyor.com/project/jvilk/bleak/branch/master)
[![npm version](https://badge.fury.io/js/bleak-detector.svg)](https://www.npmjs.com/package/bleak-detector)
[![Coverage Status](https://coveralls.io/repos/github/plasma-umass/BLeak/badge.svg)](https://coveralls.io/github/plasma-umass/BLeak)

[苍凉](http://bleak-detector.org/)在Web应用程序的客户端自动查找,排名和诊断内存泄漏.

BLeak使用一个简短的开发人员提供的脚本来循环驱动应用程序通过特定的可视状态(例如,邮件客户端的收件箱视图和电子邮件视图)作为查找内存泄漏的oracle.根据我们的经验,BLeak的精确度通常是**100%**(例如,没有误报),并且修复它发现的泄漏会减少堆增长**94%**平均而言,真实生产网络应用程序的语料库.

有关详细信息,请参阅[BLeak网站](http://bleak-detector.org/)和[我们的学术论文](https://github.com/plasma-umass/BLeak/blob/master/paper.pdf),出现在PLDI 2018年.

## 先决条件

必须安装以下内容才能使BLeak工作:

-   [mitmproxy](https://mitmproxy.org/)V4(经4.0.1测试)
-   Python 3.6或更高版本
    -   我们的mitmproxy插件使用新的Python异步功能

## 安装

```
npm install -g bleak-detector
```

安装后,您应该能够运行`bleak`从命令行.

## 运用

1.  **建立**BLeak(见上文).
2.  **写**一个*配置文件*为您的Web应用程序(见下文).
3.  **跑** `bleak run --config path/to/config.js --out path/to/where/you/want/output`
    -   对于此特定的BLeak运行,输出目录应该是唯一的,否则它将覆盖目录中的文件.如果需要,它将被创建.
4.  **等待.**BLeak通常在\<10分钟内运行,但其速度取决于循环中的状态数和Web应用程序的速度.
5.  **运行BLeak结果查看器**通过运行`bleak viewer`并导航到[HTTP://本地主机:8889 /](http://localhost:8889/)在Web浏览器中.上传`path/to/where/you/want/output/bleak_results.json`到Web应用程序查看结果!
    -   或者,BLeak打印出一份报告`bleak_report.log`在同一目录中,但结果查看器显示该日志文件中未捕获的其他信息.

## 配置文件

BLeak使用配置文件在Web应用程序的客户端查找内存泄漏.只需要几个字段.

```javascript
// URL to the web application.
exports.url = "http://path/to/my/site";
// Runs your program in a loop. Each item in the array is a `state`. Each `state` has a "check"
// function, and a "next" function to transition to the next state in the loop. These run
// in the global scope of your web app.
// BLeak assumes that the app is in the first state when it navigates to the URL. If you specify
// optional setup states, then it assumes that the final setup state transitions the web app to
// the first state in the loop.
// The last state in the loop must transition back to the first.
exports.loop = [
  // First state
  {
    // Return 'true' if the web application is ready for `next` to be run.
    check: function() {
      // Example: `group-listing` must be on the webpage
      return !!document.getElementById('group-listing');
    },
    // Transitions to the next state.
    next: function() {
      // Example: Navigate to the first thread
      document.getElementById("thread-001").click();
    }
  },
  // Second (and last) state
  {
    check: function() {
      // Example: Make sure the body of the thread has loaded.
      return !!document.getElementById('thread-body');
    },
    // Since this is the last state in the loop, it must transition back to the first state.
    next: function() {
      // Example: Click back to group listing
      document.getElementById('group-001').click();
    }
  }
];

// (Optional) Number of loop iterations to perform during leak detection (default: 8)
exports.iterations = 8;

// (Optional) An array of states describing how to login to the application. Executed *once*
// to set up the session. See 'config.loop' for a description of a state.
exports.login = [
  {
    check: function() {
      // Return 'true' if the element 'password-field' exists.
      return !!document.getElementById('password-field');
    },
    next: function() {
      // Log in to the application.
      const pswd = document.getElementById('password-field');
      const uname = document.getElementById('username-field');
      const submitBtn = document.getElementById('submit');
      uname.value = 'spongebob';
      pswd.value = 'squarepants';
      submitBtn.click();
    }
  }
];
// (Optional) An array of states describing how to get from config.url to the first state in
// the loop. Executed each time the tool explicitly re-navigates to config.url. See
// config.loop for a description of states.
exports.setup = [

];
// (Optional) How long (in milliseconds) to wait for a state transition to finish before declaring an error.
// Defaults to 10 minutes
exports.timeout = 10 * 60 * 1000;
// (Optional) How long (in milliseconds) to wait between a check() returning 'true' and transitioning to the next step or taking a heap snapshot.
// Default: 1000
exports.postCheckSleep = 1000;
// (Optional) How long (in milliseconds) to wait between transitioning to the next step and running check() for the first time.
// Default: 0
exports.postNextSleep = 0;
// (Optional) How long (in milliseconds) to wait between submitting login credentials and reloading the page for a run.
// Default: 5000
exports.postLoginSleep = 5000;
// (Optional) An array of numerical IDs identifying leaks with fixes in your code. Used to
// evaluate memory savings with different leak configurations and the effectiveness of bug fixes.
// In the code, condition the fix on $$$SHOULDFIX$$$(ID), or add logic to `exports.rewrite` (see below),
// and BLeak will run the web app with the fixes applied.
exports.fixedLeaks = [0, 1, 2];
// (Optional) Proxy re-write rule that runs in a Node.js environment, *not* in the browser.
// Lets you rewrite the web app's JavaScript/HTML/CSS to test bug fixes. Especially useful for evaluating
// fixes on web apps you do not control.
// Return a Node.js Buffer containing the replacement resource contents, or the original contents if not
// modifying.
exports.rewrite = function(url /* URL of the resource */,
                  type /* MIME type of resource */,
                  data /* Contents of resource, as a Node.js Buffer */,
                  fixes /* Array of numerical IDs corresponding to bug fixes that are active during the session (see fixedLeaks) */) {
  function hasFix(n) {
    return fixes.indexOf(n) !== -1;
  }
  // Example: Filter out non-JavaScript resources.
  if (type.indexOf("javascript") !== -1) {
    if (url.indexOf("19/common.js") !== -1) {
      let src = data.toString();
      // Example: Replace a specific string in `19/common.js` to fix bug 0.
      if (hasFix(0)) {
        src = src.replace(`window.addEventListener("scroll",a,!1)`, 'window.onscroll=a');
      }
      return Buffer.from(src, 'utf8');
    }
  }
  return data;
};
```

## 发展

有兴趣修复bug或在BLeak上修建?优秀!请阅读以下有关如何从源代码构建BLeak并运行我们的单元测试的信息.

### 先决条件

-   [纱](https://yarnpkg.com/en/docs/install)包经理
    -   NPM*可以*工作,但我们不反对它

### 建造

```
# Install NPM dependencies (only need to run once)
yarn install
# Build BLeak
yarn run build
```

### 测试

```
yarn test
```

### 调试技巧

凄凉的可执行文件(可通过运行`./bleak`一旦构建)有许多有用的调试命令.例如,使用`proxy-session`调试BLeak的代理/诊断阶段的问题.
