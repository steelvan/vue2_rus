
# Примеси

[Содержание учебника](../index.html)

## Основы

Примеси (mixins) — это гибкий инструмент повторного использования кода в компонентах Vue. Объект примеси может содержать любые настройки компонентов. При использовании компонентом примеси, все настройки примеси «подмешиваются» к собственным настройкам компонента.

Пример:

```js
// определяем объект примеси
var myMixin = {
  created: function () {
    this.hello()
  },
  methods: {
    hello: function () {
      console.log('привет из примеси!')
    }
  }
}

// определяем компонент, использующий примесь
var Component = Vue.extend({
  mixins: [myMixin]
})

var component = new Component() // => "привет из примеси!"
```

## Слияние настроек

Если примесь и компонент содержат пересекающиеся настройки, они будут определённым образом объединены.

Например, объект `data` претерпевает рекурсивное слияние с объектом `data` компонента, который имеет приоритет в случаях конфликтов.

```js
var mixin = {
  data: function () {
    return {
      message: 'hello',
      foo: 'abc'
    }
  }
}

new Vue({
  mixins: [mixin],
  data: function () {
    return {
      message: 'goodbye',
      bar: 'def'
    }
  },
  created: function () {
    console.log(this.$data)
    // => { message: "goodbye", foo: "abc", bar: "def" }
  }
})
```

Функции перехватчиков с одинаковыми именами объединяются в массив, чтобы все они вызывались. Перехватчики примеси будут вызываться **перед** собственными перехватчиками компонента.

```js
var mixin = {
  created: function () {
    console.log('вызван перехватчик примеси')
  }
}

new Vue({
  mixins: [mixin],
  created: function () {
    console.log('вызван перехватчик компонента')
  }
})

// => "вызван перехватчик примеси"
// => "вызван перехватчик компонента"
```

Настройки,  ожидающие значения в форме объектов, такие как `methods`, `components` и `directives` будут объединены. В случае конфликта, приоритет имеют настройки компонента:

```js
var mixin = {
  methods: {
    foo: function () {
      console.log('foo')
    },
    conflicting: function () {
      console.log('из примеси')
    }
  }
}

var vm = new Vue({
  mixins: [mixin],
  methods: {
    bar: function () {
      console.log('bar')
    },
    conflicting: function () {
      console.log('из самого компонента')
    }
  }
})

vm.foo() // => "foo"
vm.bar() // => "bar"
vm.conflicting() // => "из самого компонента"
```

Обратите внимание, что те же самые стратегии слияния используются и во `Vue.extend()`.

## Глобальные примеси

Примесь может быть применена и глобально. Но используйте данную возможность осторожно! После применения, примесь окажет влияние на **все** экземпляры Vue, создаваемые в дальнейшем. При правильном использовании это можно использовать для вставки логики обработки пользовательских настроек:

```js
// добавляем обработчик для пользовательской настройки `myOption`
Vue.mixin({
  created: function () {
    var myOption = this.$options.myOption
    if (myOption) {
      console.log(myOption)
    }
  }
})

new Vue({
  myOption: 'hello!'
})
// => "hello!"
```

Используйте глобальные примеси с осторожностью, поскольку они влияют на все до единого создаваемые экземпляры Vue, включая внешние компоненты. В большинстве случаев их стоит использовать только для обработки пользовательских настроек, подобно примеру выше. Неплохой идеей будет их оформление в виде [добавок](plugins.md), что позволит избежать дублирования кода.

## Пользовательские стратегии слияния настроек

При слиянии пользовательских настроек применяется стратегия по умолчанию, которая просто заменяет одни значения другими. Если вы хотите использовать пользовательскую логику при слиянии пользовательских настроек, добавьте функцию в `Vue.config.optionMergeStrategies`:

```js
Vue.config.optionMergeStrategies.myOption = function (toVal, fromVal) {
  // return mergedVal
}
```

Для большей части настроек-объектов можно просто использовать стратегию, применяемую по умолчанию для настройки `methods`:

```js
var strategies = Vue.config.optionMergeStrategies
strategies.myOption = strategies.methods
```

Более сложным примером может послужить стратегия слияния из [Vuex](https://github.com/vuejs/vuex) 1.x:

```js
const merge = Vue.config.optionMergeStrategies.computed
Vue.config.optionMergeStrategies.vuex = function (toVal, fromVal) {
  if (!toVal) return fromVal
  if (!fromVal) return toVal
  return {
    getters: merge(toVal.getters, fromVal.getters),
    state: merge(toVal.state, fromVal.state),
    actions: merge(toVal.actions, fromVal.actions)
  }
}
```
