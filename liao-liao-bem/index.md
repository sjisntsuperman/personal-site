# BEM


<!--more-->

## 前言

用了很久的CSS，一直听说要使用BEM规范。这样写起来的CSS才不那么像坨翔。因此来回顾一下BEM规范。

## Block

指的是一个功能块，举个例子，

```css
.game{}
.form{}
```

这样就是一个大块，里面还有很多的相关联的 小块。这时候要用到element。

## Element

指的是与功能块相关联的样式块，或者是说构成功能块里面的小功能块。举个例子，

```css
.game__header{}
.form__submit{}

/* less */
.game{
    &__header{}
}

.form{
    &__submit{}
}
```

这样用某些预编译器能更好的耦合起来。

## Modifier

指的是某一样式块的状态。比如一些开关，

```css
.game{
    &__header{
        &--active{}
        &--disabled{}
    }
}

.form{
    &__submit{
        &--active{}
        &--disabled{}
    }
}

/* 也有这样的 */
.game{
    &__header{
        .active{}
        .disabled{}
    }
}
```

上面那种明显更加解耦一点。看起来也会舒服点。

## Reference

1. <http://getbem.com/naming/>
2. <https://css-tricks.com/bem-101/>

