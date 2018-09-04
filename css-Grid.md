
## History of Layout
tables -> positioning -> inline-block： 遗留问题： 垂直居中
flexbox 基本可以解决横向和纵向布局的问题，但是它是为一维布局设置的，`flex-directions: rows / colums` 布局容器只能设置为一种属性。
css grid： 是第一个专门用于解决布局问题的css模块。
## What is CSS Grid ?
>  A two-dimensional grid-based layout system that aims to do nothing less than completely change the way we designed grid-based user interfaces.

二维的基于栅格的布局系统，完全改变了我们设计基于栅格用户界面的设计。
## 浏览器支持
当使用新的css样式的时候，浏览器是否支持已经成为我们必须要解决的问题之一。当css 用于布局的时候我们考虑的更多，例如 Flexbox 或者 CSS Grid。

## 基础用法
flexbox: flexbox 允许你在单一方向设置css规则，flex-direction: columns / rows
cssGrid 允许你同时在两个方向设置规则，并且提供更深层次的专为布局设置的规则

用法：
1. `CSS GRID` 用于 布局 (Layout Level)
2. `FLEXBOX` 用于具体组件排列(Component Level) (aranging child items of components)

这个用法是适用于大多数应用场景，当然，在某些功能方面他们是有一定的重叠，所以在某些场景他们的用法是可以互换的。

## CSS Grid 布局基础

### sass mixin
为保持所有的 `grid` 实例一致，我们先来创建一个 Sass mixin
````css
 @mixin display-grid {
     display: grid;
     grid-template-rows: auto;
     grid-gap: 1.5em;
 }
````

在 `container` 级别的 `sections` 中引入：

````css
.container {
    @include display-grid;
    grid-template-columns: repeat(2, 1fr);
}
````

下面是一个包含窗口大小的具体例子:
````css
.container {
    @include media-breakpint-up(md) {
        @include display-grid;
        grid-template-columns: repeat(2, 1fr);
    }

     @include media-breakpint-up(lg) {
        @include display-grid;
        grid-template-columns: repeat(4, 1fr);
    }
}
````

没有多余的依赖，所有的样式都在浏览器内。

### 重复利用样式
在实际开发中，我们可以创建通过的类，通过这些类可以比较容易的在`container`组件中应用容器：
````css

%grid-helpers-base {
    @include media-breakpoint-up(md) {
        @include display-grid;
        grid-template-columns: 1fr;
    }
}

.cols-2 {
    @extend %grid-helpers-base;
    @include media-breakpoint-up(md) {
        grid-template-columns: repeat(2, 1fr);
    }
}
.cols-3 {
    @extend %grid-helpers-base;
    @include media-breakpoint-up(md) {
        grid-template-columns: repeat(3, 1fr);
    }
}
.cols-4 {
    @extend %grid-helpers-base;
    @include media-breakpoint-up(md) {
        grid-template-columns: repeat(4, 1fr);
    }
}
.cols-auto-fit {
    @extend %grid-helpers-base;
    @include media-breakpoint-up(md) {
        grid-template-columns: repeat(auto-fit, minmax(15.625em, 1fr));
    }
}

````
这些基本可以帮助我们解决 `75%` 的应用场景。剩下的需要我们创建自己的 `css` 样式。




### 浏览器回退

我们选择[Modernizr](https://modernizr.com/) 作为回退方法。

当 `CSS Grid` `Modernizr` 可以在页面的 `html` 标签添加 `.no-cssgrid` 样式。这样允许我们编写所有浏览器都支持的内嵌规则，如浮动：
````css
.no-cssgrid {
    .cols-2 {
        @include media-breakpoint-up(md) {
            >* {
                width: 50%;
                padding-right: $baseline*3;
                float: left;
                margin-bottom: $baseline*3;
            }
        }
    }
    .cols-3 {
        @include media-breakpoint-up(md) {
            >* {
                width: 33.3%;
                padding-right: $baseline*3;
                float: left;
                margin-bottom: $baseline*3;
            }
        }
    }
    .cols-4 {
        @include media-breakpoint-up(md) {
            >* {
                width: 25%;
                padding-right: $baseline*3;
                float: left;
                margin-bottom: $baseline*3;
            }
        }
    }
    .cols-auto-fit {
        @include media-breakpoint-up(md) {
            text-align: center;
            >* {
                min-width: 15.625em;
                padding-right: $baseline*3;
                display: inline-block;
                margin-bottom: $baseline*3;
            }
        }
    }
}
````
可以在google 的代理上检测是是移动端友好 [mobile friendly test](https://search.google.com/test/mobile-friendly)。

在 CSS Grid 中规则不会对没有支持的浏览器产生任何影响。

### 总结
CSS Grid

## 参考
1. [How We Adopted CSS Grid at Scale?](https://julian.is/article/css-grid-at-scale/)
2. [CSS Grid 示例和模式](https://gridbyexample.com/)
