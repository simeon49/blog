+++
title = "web-component"
description = "web-component"
tags = [
    "js", "web component"
]
date = "2019-04-12"
categories = [
    "web",
]
menu = "main"
+++

[w3c web components](https://w3c.github.io/webcomponents/)

## 什么是web component

前端框架已经变得很流行, Angular, Vue, React... 这些框架为开发者提供很多便于使用方法来解决实际问题, 组件就是这些技术中的一个, 许多人为框架编写组件来减少自己和他人的开发工作量, 框架的组件使用起来很方便但问题是使用不同框架开发的组件不能用于其它框架, 于是W3C做了一个决定让浏览器直接支组件这一特性(持按某一规范编写). 这就是 Web Components 通过一种标准化的非侵入的方式封装一个组件，每个组件能组织好它自身的 HTML 结构、CSS 样式、JavaScript 代码，并且不会干扰页面上的其他代码<br>

## 涉及到的技术

#### 自定义元素（Custom Elements）

Custom Elements 顾名思义，是提供一种方式让开发者可以自定义 HTML 元素，包括特定的组成，样式和行为。支持 Web Components 标准的浏览器会提供一系列 API 给开发者用于创建自定义的元素，或者扩展现有元素。

#### 影子DOM（Shadow DOM）

影子DOM API提供了attachShadow()方法，创建一个影子DOM，支持将封装的内容或组件作为一个独立DOM子树附加进一个HTML文档，组件内与外部隔离，样式互不影响，这也印证了组件开发的封装性需求

#### HTML导入（HTML Imports）

如何在HTML文档中引入另一个web文档或web组件呢？像JSP或PHP语言都对HTML语法进行了拓展，我们可以使用诸如<include>标签直接引入另一个文档，然而在这之前，原生HTML规范并不支持直接引入另一文档，通常都得通过ajax请求另一文档内容，然后通过JavaScript使用DOM API将内容插入，对于组件化开发和使用，这样显然不是我们期望的结果，这与组件的易用性是背离的，所以，HTML imports定义了如何在文档内引入和重用另一文档。

在文档内直接引入外链资源的文档或web组件，语法如下，使用<link>标签：

```html
<link rel="import" href="components.html">
```

#### HTML模板（HTML Templates）

为了更友好的处理组件模板，Web Components规范，支持\<template\>模板标签，HTML模板定义了使用\<template\>标签声明可以通过脚本操作插入文档的HTML模板片段：

```html
<template id="menusTemplate">
    <ul>
        <li>Home</li>
        <li>About</li>
    </ul>
</template>
```

使用脚本操作，该元素content属性可访问模板内容：

```js
var menusTemplate = document.querySelector('#menusTemplate');
var frag = document.importNode(menusTemplate.content, true);
document.querySelector('.menus').appendChild(frag);
```

> **TEMPLATE标签:**
> \<template\>标签本质上与其他HTML内置标签一样，可以使用DOM API进行操作，但是需要明白，在将模板激活（生成DOM或插入文档）前：
> 1. \<template\>标签内的内容不会被渲染；
> 2. 标签内的图片，等媒体资源不会被加载；
> 3. 标签不会出现在DOM树，审查元素看不到;


## 实例

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <script src="./components/components.js"></script>
  <style>
    /* 选择所有已定义的元素 */
    :defined {
      font-style: italic;
    }
    /* 全局的样式将不会对 national-flag 里的样式产生影响 */
    img {
      width: 500px;
      height: 500px;
      background: red;
      display: block;
    }
    .flag {
      text-align: center;
      user-select: none;
    }
    label {
      cursor: pointer;
    }
  </style>
  <title>Web Component</title>
</head>
<body>
  <div class="flag">
    <national-flag id="national-flag" country='America'>
      <p>国旗</p> <!-- 此处代码将在 slot 位置显示 -->
    </national-flag><br>

    <p id="country-choice">选择国家旗帜:
      <label><input type="radio" name="country" value="America" checked="checked">America</label>
      <label><input type="radio" name="country" value="England">England</label>
      <label><input type="radio" name="country" value="France">France</label>
      <label><input type="radio" name="country" value="*">Other</label>
    </p>
  </div>

  <script>
    document.getElementById('country-choice').addEventListener('click', ev => {
      window['ev'] = ev;
      if (ev.target.nodeName.toLowerCase() === 'input' && ev.target.name === 'country') {
        document.getElementById('national-flag').country = ev.target.value;
      }
    })
  </script>
</body>
</html>

```

```js
const _FLAGS_PATH_MAP = {
  'america': './components/assert/America.svg',
  'england': './components/assert/England.svg',
  'france': './components/assert/France.svg',
  'unknown': './components/assert/Unknown.svg',
};


class NationalFlag extends HTMLElement {
  constructor() {
    super();
    this.isInint = false;
  }

  connectedCallback() {
    console.log('connectedCallback is called.');
    const shadowRoot = this.attachShadow({mode: 'open'});
    shadowRoot.innerHTML = `
    <style>
      img {
        width: 128px;
        height: 128px;
      }
    </style>

    <img src="${this._getCountryFlag(this.country)}" />
    <slot><slot>
    `;
    this.isInint = true;
  }

  disconnectedCallback() {
    console.log('disconnectedCallback is called.');
  }

  get country() {
    return this.getAttribute('country');
  }

  set country(country) {
    console.log(`set country: ${country}`);
    this.setAttribute('country', country);
    const img = this.shadowRoot.querySelector('img');
    img.src = this._getCountryFlag(this.country);
  }

  _getCountryFlag(country) {
    country = country.toLowerCase();
    let path = _FLAGS_PATH_MAP[country];
    path = path ? path : _FLAGS_PATH_MAP.unknown;
    return path;
  }
}
// 定义 national-flag 标签
customElements.define('national-flag', NationalFlag);

```

直接看[源码](https://github.com/simeon49/javascript-practices/tree/master/project_10_web_component) <br>
直接看[Demo](https://simeon49.github.io/javascript-practices/project_10_web_component/index.html) <br>
