---
title: CSS命名规范----节省调试时间，便于后期维护必读
tags: CSS
---

## 1、避免使用驼峰式命名法（camel case），用连字符分隔法（hyphen delimiters）代替
**js**中驼峰式命名**非常常见**，但在CSS中使用**更易读**的连字符分隔方式才是王道，这也和 CSS **属性名称**保持了一致

<!--more-->

```
// bad
.redBox {
  ...
}

// good
.red-box {
  ...
}
```
---
## 2、如果你的项目用户界面更复杂，建议使用 BEM 命名法
一般情况连字符命名法已经适用大部分情况了，但在界面更复杂的情况中，用仅从名称就可以获取：
- 选择器具体做什么
- 选择器大概在哪使用
- 选择器之间大概联系

大量信息的BEM命名法，将是更好的选择。
#### BEM命名法
Bem 是块（block）、元素（element）、修饰符（modifier）的简写，由 Yandex 团队提出的一种前端 CSS 命名方法论。


【-】 中划线 ：仅作为连字符使用，表示某个块或者某个子元素的多单词之间的连接记号。
```
.stick-man { }
```

【__】 双下划线：双下划线用来连接块和块的子元素
```
.stick-man__head { }
```

【--】  双中划线：用于连接修饰符
```
.stick-man--blue { }
```
---
## 用 js- 类名前缀，表示类名和js有关联
防止出现误操作情况，导致js代码失效报错问题
```
// bad
<div class="site-navigation"></div>
<script>
    const nav = document.querySelector('.site-navigation')
</script>

// good
<div class="js-site-navigation"></div>
<script>
    const nav = document.querySelector('.js-site-navigation')
 </script>
```
---
## 为你的复杂css代码添加详细备注
### 参考
[1] [bootstrap前端框架](https://github.com/twbs/bootstrap/blob/v4-dev/scss/_carousel.scss)