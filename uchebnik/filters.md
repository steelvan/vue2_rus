
# Фильтры


{% raw %}
<script src="https://cdn.jsdelivr.net/npm/vue@2.6.12"></script>
<style>
.demo{
  border: 1px solid #eee;
  border-radius: 2px;
  padding: 25px 35px;
  margin-top: 1em;
  margin-bottom: 40px;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  overflow-x: auto;    
}
</style>
{% endraw %}

Для распространённых задач форматирования текста во Vue используются фильтры. Они находят своё применение в двух местах: **в mustache-интерполяциях и в выражениях `v-bind`** (последнее поддерживается в 2.1.0+). Фильтры добавляются в конце выражения JavaScript и отделяются вертикальной чертой:

{% raw %}
```html
<!-- mustache -->
{{ message | capitalize }}

<!-- v-bind -->
<div v-bind:id="rawId | formatId"></div>
```
{% endraw %}

Вы можете определить местные фильтры в настройках компонента:

```js
filters: {
  capitalize: function (value) {
    if (!value) return ''
    value = value.toString()
    return value.charAt(0).toUpperCase() + value.slice(1)
  }
}
```

или определить фильтр глобально перед созданием экземпляра Vue:

```js
Vue.filter('capitalize', function (value) {
  if (!value) return ''
  value = value.toString()
  return value.charAt(0).toUpperCase() + value.slice(1)
})

new Vue({
  // ...
})
```

Когда у глобального фильтра такое же имя как и у местного, то местный фильтр будет приоритетнее.

Ниже приведён пример использования нашего фильтра `capitalize`:

{% raw %}
<div id="example_1" class="demo">
  <input type="text" v-model="message">
  <p>{{ message | capitalize }}</p>
</div>
<script>
  new Vue({
    el: '#example_1',
    data: function () {
      return {
        message: 'john'
      }
    },
    filters: {
      capitalize: function (value) {
        if (!value) return ''
        value = value.toString()
        return value.charAt(0).toUpperCase() + value.slice(1)
      }
    }
  })
</script>
{% endraw %}

Функция фильтра всегда принимает значение выражения (результат предыдущей цепочки) в качестве первого аргумента. В этом примере функция фильтра `capitalize` получит значение `message` в качестве аргумента.

Фильтры можно объединять в цепочки:

{% raw %}
```html
{{ message | filterA | filterB }}
```
{% endraw %}

В этом случае `filterA`, определённый с одним аргументом, получит значение `message`, а затем функция `filterB` будет вызвана с результатом `filterA`, переданным единственным аргументом в `filterB`.

Фильтры являются JavaScript-функциями, и потому могут принимать параметры:

{% raw %}
```html
{{ message | filterA('arg1', arg2) }}
```
{% endraw %}

В данном случае `filterA` определяется как функция с тремя аргументами. Значение `message` будет передано первым аргументом. Простая строка `'arg1'` будет передана в `filterA` вторым аргументом, а значение выражения `arg2` — третьим.
