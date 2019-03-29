# plasma-umass/BLeak [![explain]][source] [![translate-svg]][translate-list]

<!-- [![size-img]][size] -->

[explain]: http://llever.com/explain.svg
[source]: https://github.com/chinanf-boy/Source-Explain
[translate-svg]: http://llever.com/translate.svg
[translate-list]: https://github.com/chinanf-boy/chinese-translate-list
[size-img]: https://packagephobia.now.sh/badge?p=Name
[size]: https://packagephobia.now.sh/result?p=Name

「 在 Web 应用程序的客户端中，自动查找,排名和诊断内存泄漏. 」

[中文](./readme.md) | [english](https://github.com/plasma-umass/BLeak)

---

## 校对 ✅

<!-- doc-templite START generated -->
<!-- repo = 'plasma-umass/BLeak' -->
<!-- commit = 'f9d3c14722e88edcda6520e57533d519b3cce405' -->
<!-- time = '2018-11-20' -->
翻译的原文 | 与日期 | 最新更新 | 更多
---|---|---|---
[commit] | ⏰ 2018-11-20 | ![last] | [中文翻译][translate-list]

[last]: https://img.shields.io/github/last-commit/plasma-umass/BLeak.svg
[commit]: https://github.com/plasma-umass/BLeak/tree/f9d3c14722e88edcda6520e57533d519b3cce405

<!-- doc-templite END generated -->

### 贡献

欢迎 👏 勘误/校对/更新贡献 😊 [具体贡献请看](https://github.com/chinanf-boy/chinese-translate-list#贡献)

## 生活

[If help, **buy** me coffee —— 营养跟不上了，给我来瓶营养快线吧! 💰](https://github.com/chinanf-boy/live-need-money)

---

# BLeak v1.2.1

[![Build Status](https://travis-ci.org/plasma-umass/BLeak.svg?branch=master)](https://travis-ci.org/plasma-umass/BLeak)
[![Build status](https://ci.appveyor.com/api/projects/status/b92sknh0pu38943q/branch/master?svg=true)](https://ci.appveyor.com/project/jvilk/bleak/branch/master)
[![npm version](https://badge.fury.io/js/bleak-detector.svg)](https://www.npmjs.com/package/bleak-detector)
[![Coverage Status](https://coveralls.io/repos/github/plasma-umass/BLeak/badge.svg)](https://coveralls.io/github/plasma-umass/BLeak)

[BLeak](http://bleak-detector.org/)会在 Web 应用程序的客户端中，自动查找,排名和诊断内存泄漏.

BLeak 使用一个简短的开发人员提供的脚本，来通过特定的可视状态,循环驱动应用程序(例如,邮件客户端的收件箱视图和电子邮件视图)作为查找内存泄漏的标杆。根据我们的经验,BLeak 的精确度通常是**100%**(例如,没有误报)，并且修复它发现的泄漏，**94%**会减少堆增长(平均而言)。对象都为真实生产网络应用程序。

有关详细信息,请参阅[BLeak 网站](http://bleak-detector.org/)和[我们的学术论文](https://github.com/plasma-umass/BLeak/blob/master/paper.pdf),出现在 PLDI 2018 年.

## 先决条件

必须安装以下内容，才能使 BLeak 工作:

- [mitmproxy](https://mitmproxy.org/)V4(4.0.1 已测试)
- Python 3.6 或更高版本
  - 我们的 mitmproxy 插件使用 Python 新的异步功能

## 安装

```
npm install -g bleak-detector
```

安装后,您应该能够在命令行运行`bleak`.

## 运用

1.  **构建-Build** BLeak(见上文).
2.  **写-Write** 一个*配置文件*，为您的 Web 应用程序(见下文).
3.  **运行-Run** `bleak run --config path/to/config.js --out path/to/where/you/want/output`
    - 对于此特定的 BLeak 运行,输出目录应该是唯一的,否则它将覆盖目录中的文件。如果需要,它将被创建.
4.  **等待.-Wait** BLeak 通常在 <10 分钟 内运行，但其速度取决于循环中的状态数和 Web 应用程序的速度.
5.  **运行 BLeak 结果查看器** 通过运行`bleak viewer`，并浏览器导航到[HTTP://本地主机:8889/](http://localhost:8889/)。上传`path/to/where/you/want/output/bleak_results.json`到 Web 应用程序查看结果!
    - 或者,BLeak 在同一目录中，打印出一份报告`bleak_report.log`,但结果查看器能显示该日志文件中未捕获的其他信息.

## 配置文件

BLeak 使用配置文件，在 Web 应用程序的客户端查找内存泄漏.只需要几个字段.

```javascript
// web 应用的 URL .
exports.url = 'http://path/to/my/site';
// 可运行你的程序，在 一个 loop . 数组中的每个对象项都是一个 `state`. 每个 `state` 有一个 "check"
// 函数, 和一个 "next" 函数 to transition to the next state in the loop. These run
// in the global scope of your web app.
// BLeak assumes that the app is in the first state when it navigates to the URL. If you specify
// 可选 setup states, then it assumes that the final setup state transitions the web app to
// the first state in the loop.
// The last state in the loop must transition back to the first.
exports.loop = [
  // 第一个 状态(state)
  {
    // 返回 'true'， 若 web 应用 已准备好`next`的运行
    check: function() {
      // 例: `group-listing` 必须在网页中
      return !!document.getElementById('group-listing');
    },
    // 过渡到下一状态
    next: function() {
      // 例: Navigate to the first thread
      document.getElementById('thread-001').click();
    }
  },
  // 第二个 (也是最后的 last) 状态
  {
    check: function() {
      // 例: 确保 ‘thread-body’ 已加载
      return !!document.getElementById('thread-body');
    },
    // 鉴于这就是loop的最后一步, 它会过渡回第一状态.
    next: function() {
      // 例: Click back to group listing
      document.getElementById('group-001').click();
    }
  }
];

// (可选) 泄漏检测期间要执行的循环迭代次数 (默认: 8)
exports.iterations = 8;

// (可选) 数组状态，指明如何登录该应用. 执行 *一次*
// 该会话. 看 'config.loop' 了解其状态的描述
exports.login = [
  {
    check: function() {
      // 返回 'true' ，若元素 'password-field' 存在.
      return !!document.getElementById('password-field');
    },
    next: function() {
      // 登入 应用.
      const pswd = document.getElementById('password-field');
      const uname = document.getElementById('username-field');
      const submitBtn = document.getElementById('submit');
      uname.value = 'spongebob';
      pswd.value = 'squarepants';
      submitBtn.click();
    }
  }
];
// (可选) An array of states describing how to get from config.url to the first state in
// the loop. Executed each time the tool explicitly re-navigates to config.url. See
// config.loop for a description of states.
exports.setup = [];
// (可选) 在报告一个错误之前，等待 一个状态完成过渡多久 (毫秒单位)
// 默认: 10 分钟
exports.timeout = 10 * 60 * 1000;
// (可选) 在 一个 check() 返回 'true' 和 过渡到下一步或拿到一个堆快照 之间，等待多久 (毫秒单位)
// 默认: 1000
exports.postCheckSleep = 1000;
// (可选) 在 过渡到下一步和第一时间运行check()之间，等待多久 (毫秒单位)
// 默认: 0
exports.postNextSleep = 0;
// (可选)  在提交登入凭证 和重载运行页面，等待多久 (毫秒单位)
// 默认: 5000
exports.postLoginSleep = 5000;
// (可选) 一组数字ID，用于识别代码中的修复漏洞. 被用来
// 作为，不同的泄漏配置和错误修复的有效内存节省的评估.
// 在代码中, 条件性修复 $$$SHOULDFIX$$$(ID), 或给 `exports.rewrite` 添加逻辑 (看下面),
// 和 BLeak 会用修复的补丁运行 web应用.
exports.fixedLeaks = [0, 1, 2];
// (可选) 重写的代理会在一个Node.js环境中运行，*不是* 在浏览器
// 让你重写 web 应用的 you rewrite the web app's JavaScript/HTML/CSS ，可测试补丁的有效性. 
// 对评估web应用中不可控，特别有用
// 返回一个包含替换资源内容的 Node.js Buffer, 或不修改的化，就返回源内容
exports.rewrite = function(
  url /*  资源 URL */,
  type /*  资源 MIME type */,
  data /*  资源 Contents, 作为 一个 Node.js Buffer */,
  fixes /* 对应于活动会话期间的错误修复的数组数字ID  (看 fixedLeaks) */
) {
  function hasFix(n) {
    return fixes.indexOf(n) !== -1;
  }
  // 例: 过滤掉 非JavaScript 的资源.
  if (type.indexOf('javascript') !== -1) {
    if (url.indexOf('19/common.js') !== -1) {
      let src = data.toString();
      // 例: 在 `19/common.js`，替换自定字符串，以修复 bug 0.
      if (hasFix(0)) {
        src = src.replace(
          `window.addEventListener("scroll",a,!1)`,
          'window.onscroll=a'
        );
      }
      return Buffer.from(src, 'utf8');
    }
  }
  return data;
};
```

## 发展

有兴趣修复 bug 或在 BLeak 上构建? 太好了! 请阅读以下有关如何从源代码构建 BLeak ，并运行我们的单元测试的信息.

### 先决条件

- [yarn](https://yarnpkg.com/en/docs/install)包管理
  - npm*可以*工作,但我们并针对它做测试

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

bleak 的可执行文件(一旦构建，可通过`./bleak`运行)有许多有用的调试命令。例如,使用`proxy-session`调试 BLeak 的代理/诊断阶段的问题.
