
# Пользовательские директивы

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

## Введение

Помимо встроенных директив (таких как `v-model` и `v-show`), Vue позволяет использовать ваши собственные. При этом важно понимать, что основным механизмом создания повторно используемого кода во Vue 2.0 всё-таки являются компоненты. Тем не менее, для выполнения низкоуровневых операций с DOM пользовательские директивы могут очень пригодиться. В качестве примера сделаем фокус на элементе input:

{% raw %}
<div id="simplest-directive-example" class="demo">
  <input v-focus>
</div>
<script>
Vue.directive('focus', {
  inserted: function (el) {
    el.focus()
  }
})
new Vue({
  el: '#simplest-directive-example'
})
</script>
{% endraw %}

После загрузки страницы этот элемент получает фокус ввода (примечание: `autofocus` не работает на мобильном Safari). Если вы никуда не кликнули с момента открытия этой главы руководства, фокус ввода и сейчас должен быть на этом элементе. Рассмотрим директиву подробнее:

```js
// Регистрируем глобальную пользовательскую директиву `v-focus`
Vue.directive('focus', {
  // Когда привязанный элемент вставлен в DOM...
  inserted: function (el) {
    // Переключаем фокус на элемент
    el.focus()
  }
})
```

Чтобы зарегистрировать директиву местно, можно передать настройку `directives` при определении компонента:

```js
directives: {
  focus: {
    // определение директивы
    inserted: function (el) {
      el.focus()
    }
  }
}
```

Теперь в шаблонах можно использовать новый атрибут `v-focus`:

```html
<input v-focus>
```

## Перехватчики

Для жизненного цикла директивы можно указать следующие перехватчики (все они вариативны):

- `bind`: вызывается однократно, при первичном связывании директивы с элементом. Здесь можно поместить код определения.

- `inserted`: вызывается после вставки связанного элемента внутрь элемента родителя (заметьте, что сам родитель может на этот момент и не принадлежать ещё основному дереву элементов).

- `update`: вызывается после обновления VNode компонента-контейнера, __но, возможно, до обновления дочерних элементов__. Значение директивы к этому моменту может измениться, а может и нет. Сравнивая текущее и прошлое значения, вы можете избежать избыточных операций (см. ниже об аргументах перехватчиков).

Подробнее мы обсудим VNodes [позднее](./render-function.md#Виртуальный-DOM), когда будем разбирать [Функции отрисовки](./render-function.md).

- `componentUpdated`: вызывается после обновления как VNode компонента-контейнера, __так и VNode его потомков__.

- `unbind`: вызывается однократно, при отвязывании директивы от элемента.

В следующем разделе мы рассмотрим аргументы, передаваемые в эти перехватчики (а именно: `el`, `binding`, `vnode` и `oldVnode`).

## Аргументы перехватчиков

В перехватчики передаются следующие параметры:

- `el`: Элемент, к которому привязана директива. Можно использовать для прямых манипуляций с DOM.
- `binding`: Объект, содержащий следующие свойства:
  - `name`: Название директивы, без указания приставки `v-`.
  - `value`: Значение, переданное в директиву. Например, для `v-my-directive="1 + 1"` значением будет `2`.
  - `oldValue`: Предыдущее переданное в директиву значение. Доступно только для перехватчиков `update` и `componentUpdated`, и передаётся независимо от того, произошло ли в действительности его изменение.
  - `expression`: Выражение-строка, переданное в директиву. Например, для `v-my-directive="1 + 1"` это будет `"1 + 1"`.
  - `arg`: Аргумент, переданный в директиву, в случае его наличия. Например, для `v-my-directive:foo` это будет `"foo"`.
  - `modifiers`: Объект, содержащий изменители, если они есть. Например, для `v-my-directive.foo.bar`, объектом изменителей будет `{ foo: true, bar: true }`.
- `vnode`: Виртуальный элемент, созданный компилятором Vue. См. [VNode API](../api/#Интерфейс-VNode) для подробностей.
- `oldVnode`: Предыдущий виртуальный элемент, доступный только для перехватчиков `update` и `componentUpdated`.

Все аргументы, кроме `el`, следует понимать как только для чтения и никогда не изменять их. В случае необходимости передать информацию между перехватчиками рекомендуем воспользоваться [dataset](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/dataset).

Пример пользовательской директивы, использующей некоторые из описанных возможностей:

```html
<div id="hook-arguments-example" v-demo:foo.a.b="message"></div>
```

```js
Vue.directive('demo', {
  bind: function (el, binding, vnode) {
    var s = JSON.stringify
    el.innerHTML =
      'name: '       + s(binding.name) + '<br>' +
      'value: '      + s(binding.value) + '<br>' +
      'expression: ' + s(binding.expression) + '<br>' +
      'argument: '   + s(binding.arg) + '<br>' +
      'modifiers: '  + s(binding.modifiers) + '<br>' +
      'vnode keys: ' + Object.keys(vnode).join(', ')
  }
})

new Vue({
  el: '#hook-arguments-example',
  data: {
    message: 'привет!'
  }
})
```

{% raw %}
<div id="hook-arguments-example" v-demo:foo.a.b="message" class="demo"></div>
<script>
Vue.directive('demo', {
  bind: function (el, binding, vnode) {
    var s = JSON.stringify
    el.innerHTML =
      'name: '       + s(binding.name) + '<br>' +
      'value: '      + s(binding.value) + '<br>' +
      'expression: ' + s(binding.expression) + '<br>' +
      'argument: '   + s(binding.arg) + '<br>' +
      'modifiers: '  + s(binding.modifiers) + '<br>' +
      'vnode keys: ' + Object.keys(vnode).join(', ')
  }
})
new Vue({
  el: '#hook-arguments-example',
  data: {
    message: 'привет!'
  }
})
</script>
{% endraw %}

### Динамические аргументы директивы

Аргументы директивы могут быть динамическими. Например, для `v-mydirective:[argument]="value"`, `argument` может обновляться в зависимости от свойства данных экземпляра компонента! Это позволит сделать пользовательские директивы более гибкими при использовании в приложении.

Допустим, необходимо создать собственную директиву, которая позволит установить элементы на странице с помощью фиксированного позиционирования. Можно создать пользовательскую директиву, где значение определяет вертикальное положение в пикселях, например так:

```html
<div id="baseexample">
  <p>Прокрутите страницу вниз</p>
  <p v-pin="200">Элемент зафиксирован в 200px от начала страницы</p>
</div>
```

```js
Vue.directive('pin', {
  bind: function (el, binding, vnode) {
    el.style.position = 'fixed'
    el.style.top = binding.value + 'px'
  }
})

new Vue({
  el: '#baseexample'
})
```

Это закрепит элемент в 200px от начала страницы. Но что если возникнет случай, когда необходимо закрепить элемент слева, а не сверху? Для этого пригодится динамический аргумент директивы, который можно определить для каждого экземпляра компонента:

```html
<div id="dynamicexample">
  <h3>Прокрутите страницу вниз</h3>
  <p v-pin:[direction]="200">Элемент зафиксирован в 200px слева страницы.</p>
</div>
```

```js
Vue.directive('pin', {
  bind: function (el, binding, vnode) {
    el.style.position = 'fixed'
    var s = (binding.arg == 'left' ? 'left' : 'top')
    el.style[s] = binding.value + 'px'
  }
})

new Vue({
  el: '#dynamicexample',
  data: function () {
    return {
      direction: 'left'
    }
  }
})
```

Результат:

{% raw %}
<iframe height="200" style="width: 100%;" class="demo" scrolling="no" title="Dynamic Directive Arguments" src="//codepen.io/team/Vue/embed/rgLLzb/?height=300&theme-id=32763&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  Посмотрите Pen <a href='https://codepen.io/team/Vue/pen/rgLLzb/'>Dynamic Directive Arguments</a> by Vue
  (<a href='https://codepen.io/Vue'>@Vue</a>) на <a href='https://codepen.io'>CodePen</a>.
</iframe>
{% endraw %}

Теперь пользовательская директива достаточно гибкая для использования в нескольких различных случаях.

## Сокращённая запись

Часто нам может потребоваться одинаковое поведение на `bind` и `update`, а остальные перехватчики не нужны. В таком случае можно использовать сокращённую запись:

```js
Vue.directive('color-swatch', function (el, binding) {
  el.style.backgroundColor = binding.value
})
```

## Передача объекта данных в директиву

В случае, если директива должна принимать несколько параметров, можно указать объект JavaScript — годится любое правильное выражение, помните?

```html
<div v-demo="{ color: 'белый', text: 'привет!' }"></div>
```

```js
Vue.directive('demo', function (el, binding) {
  console.log(binding.value.color) // => "белый"
  console.log(binding.value.text)  // => "привет!"
})
```
