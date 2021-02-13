### kuku

## Базовый пример

Вот пример компонента Vue:

```js
// Определяем новый компонент, названный button-counter
Vue.component('button-counter', {
  data: function () {
    return {
      count: 0
    }
  },
  template: '<button v-on:click="count++">Счётчик кликов — {{ count }}</button>'
})
```
