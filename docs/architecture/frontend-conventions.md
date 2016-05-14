# 前端规范
在实际开发中，由于团队成员编码习惯不一，技术层次不同，开发前定制并遵循一种代码规范能提高代码质量，增加开发效率。

## Javascript
Javascript规范直接参考`airbnb`:

[ES6 JavaScript Style Guide](https://github.com/airbnb/javascript)

[ES5 JavaScript Style Guide](https://github.com/airbnb/javascript/tree/master/es5)

[React/JSX Style Guide](https://github.com/airbnb/javascript/tree/master/react)

## CSS
### [BEM](http://getbem.com/)
`Block Element Modifier`，它是一种前端命名方法，旨在帮助开发者实现模块化、可复用、高可维护性和结构化的CSS代码。

`BEM`是定义了一种`css class`的命名规范，每个名称及其组成部分都是存在一定的含义。

| Block | Element | Modifier |
|:------|:--------|:---------|
| 独立且有意义的实体, e.g. `header`, `container`, `menu`, `checkbox`, etc. | Block的一部分且没有独立的意义, e.g. `header title`, `menu item`, `list item`, etc. | Blocks或Elements的一种标志，可以用它改变其表现形式、行为、状态. e.g. `disabled`, `checked`, `fixed`, etc. |

#### Naming
由拉丁字母, 数字, `-`组成css的单个名称.

**Block**

使用简洁的前缀名字来命名一个独立且有意义的抽象块或组件。

e.g.

`.block`

`.header`

`.site-search`

**Element**

使用`__`连接符来连接`Block` 和 `Element`。

e.g.

`.block__element`

`.header__title`

`.site-search__field`

**Modifier**

使用`--`连接符来连接`Block` 或 `Element` 和 `Modifier`。

e.g.

`.block--modifier`

`.block__element--modifier`

`.header--hide`

`.header__title--color-red`

`.site-search__field--disabled`

#### 实例
**HTML**
```html
<form class="form form--theme-xmas form--simple">
  <input class="form__input" type="text" />
  <input class="form__submit form__submit--disabled" type="submit" />
</form>
```

**CSS**
```css
.form {
}
.form--theme-xmas {
}
.form--simple {
}
.form__input {
}
.form__submit {
}
.form__submit--disabled {
}
```

#### Buttons实例
![buttons](../../public/img/architecture/buttons.jpg)

**HTML**
```html
<button class="button">
  Normal button
</button>
<button class="button button--state-success">
  Success button
</button>
<button class="button button--state-danger">
  Danger button
</button>
```

**CSS**
```css
.button {
	display: inline-block;
	border-radius: 3px;
	padding: 7px 12px;
	border: 1px solid #D5D5D5;
	background-image: linear-gradient(#EEE, #DDD);
	font: 700 13px/18px Helvetica, arial;
}
.button--state-success {
	color: #FFF;
	background: #569E3D linear-gradient(#79D858, #569E3D) repeat-x;
	border-color: #4A993E;
}
.button--state-danger {
	color: #900;
}
```

#### FAQ
[BEM - FAQ](http://getbem.com/faq/)

### [OOCSS](https://github.com/stubbornella/oocss/wiki)
`Object Oriented CSS`，面向对象的CSS，旨在编写高可复用、低耦合和高扩展的CSS代码。

`OOCSS`是以面向对象的思想去定义样式，将抽象和实现分离，抽离公共代码。

#### 区分结构和样式
在定义一个可重用性的组件库时，我们仅创建基础的结构（html）和基础的类名，不应该创建类似于`border, width, height, background`等样式规则，这样使组件库更灵活和可扩展性。组件库在不同环境下的样式所要求不一样，若未能区分其结构和样式，给其添加样式，会使其变成一个特定的组件库，而难以重用。

e.g.

以下是一个基础库创建的样式：

```css
.metadata{
  font-size: 1.2em;
  text-align: left;
  margin: 10px 0;
}
```

若在给其添加更多的样式：

```css
.metadata{
	font-size: 1.2em;
	margin: 10px 0;
	/*在基础组件上新加的样式*/
	width: 500px;
	background-color: #efefef;
	color: #fff;
}
```

这样就使前面创建的基础组件`metadata`变成了一个特定的组件了，使其在其他场景下较难复用。

#### 区分容器和内容
把容器和内容独立分区，使内容能作用于任何容器下。

e.g.

```css
#sidebar h3 {
  font-family: Arial, Helvetica, sans-serif;
  font-size: .8em;
  line-height: 1;
  color: #777;
  text-shadow: rgba(0, 0, 0, .3) 3px 3px 6px;
}
```

上面我们定义了一个id为`sidebar` 中 `h3`的样式，但是我们发现在`footer` 中 `h3`的样式也基本一致，仅个别不一样，那么我们可能会这样写样式：

```css
#sidebar h3, #footer h3 {
  font-family: Arial, Helvetica, sans-serif;
  font-size: 2em;
  line-height: 1;
  color: #777;
  text-shadow: rgba(0, 0, 0, .3) 3px 3px 6px;
}

#footer h3 {
  font-size: 1.5em;
  text-shadow: rgba(0, 0, 0, .3) 2px 2px 4px;
}
```

甚至我们可能会用更糟糕的方式来写这个样式：

```css
#sidebar h3 {
  font-family: Arial, Helvetica, sans-serif;
  font-size: 2em;
  line-height: 1;
  color: #777;
  text-shadow: rgba(0, 0, 0, .3) 3px 3px 6px;
}

#footer h3 {
  font-family: Arial, Helvetica, sans-serif;
  font-size: 1.5em;
  line-height: 1;
  color: #777;
  text-shadow: rgba(0, 0, 0, .3) 2px 2px 4px;
}
```

我们可以看到上面的代码中出现了不必要的`duplicating styles`。而`OOCSS`鼓励我们应该思考在不同元素中哪些样式是通用的，然后将这些通用的样式从模块、组件、对象等中抽离出来，使其能在任何地方能够复用，而不依赖于某个特定的容器。
```css
.title-heading {
  font-family: Arial, Helvetica, sans-serif;
  font-size: 2em;
  line-height: 1;
  color: #777;
  text-shadow: rgba(0, 0, 0, .3) 3px 3px 6px;
}
#footer .title-heading {
  font-size: 1.5em;
  text-shadow: rgba(0, 0, 0, .3) 2px 2px 4px;
}
```

### [SMACSS](https://smacss.com/)
`Scalable and Modular Architecture for CSS`，可扩展模块化的CSS，它的核心就是结构化CSS代码，提出了一种CSS分类规则，分为五种类型：

- Base
- Layout
- Module
- State
- Theme

`SMACSS`定义了一种css文件的组织方式，其横向的切分，使css文件更具有结构化、高可维护性。

#### Base
Base是默认的样式，是对单个元素选择器（包括其属性选择器，伪类选择器，孩子/兄弟选择器）的基础样式设置，例如html, body, input[type=text], a:hover, etc.

e.g.

```css
html, body {
  margin: 0;
  padding: 0;
}

input[type=text] {
  border: 1px solid #999;
}

a {
  color: #039;
}

a:hover {
  color: #03C;
}
```

`CSS Reset/Normalize`就是一种`Base Rule`的应用.

<mark>Note:</mark>

- 在基础样式中不应该使用`!important`

#### Layout
不明思议，是对页面布局元素（页面架构中主要和次要的组件）的样式设置，例如header, navigation, footer, sidebar, login-form, etc.

e.g.

```css
.header, footer {
  margin: 0;
}

.header {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 9999;
}

.navs {
  display: inline-block;
  margin: 0 auto;
}
```

#### Modules
对公共组件样式的设置，例如dropdown, tabs, carousels, dialogs, etc.

e.g.

```css
.dropdown, .dropdown > ul {
  display: inline-block;
  border: 1px solid #283AE2;
}

.dropdown li:hover {
  background-color: #999;
}

.tabs {
  border: 1px solid #e8e8e8;
}

.tabs > .tab--active {
  border-bottom: none;
  color: #29A288;
}
```

#### State
对组件、模块、元素等表现行为或状态的样式修饰，例如hide, show, is-error, etc.

e.g.

```css
.hide {
  display: none;
}

.show {
  display: block;
}

.is-error {
  color: red;
}
```

#### Theme
对页面整体布局样式的设置，可以说是一种`皮肤`，它可以在特定场景下覆盖`base`, `layout`等的默认样式。

e.g.

```css
.body--theme-sky {
  background: blue url(/static/img/body--theme-sky.png) repeat;
}

.footer--theme-sky {
  background-image: blue url('/static/img/header--theme-sky.png') no-repeat center;
}
```

### Others
- [SUITCSS](http://suitcss.github.io/)
- [Atomic](https://github.com/nemophrost/atomic-css)
- [Airbnb CSS Style Guide](https://github.com/airbnb/css)

### [Web Components](http://webcomponents.org/)
这么多CSS规范，貌似还是有冲突等问题，咋办呀？

`世上没有完美方案，只有合适/最佳方案～`

让我门一起期待[Web Components](http://webcomponents.org/)到来吧～

## 资源
### [在线实例](http://ipluser.github.io/speechless/public/view/architecture/frontend-conventions.html)
### [源代码](https://github.com/ipluser/speechless/blob/gh-pages/public/view/architecture/frontend-conventions.html)
