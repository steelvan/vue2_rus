
# Анимование переходов между состояниями

[Содержание учебника](../index.html)

Система анимования переходов Vue предоставляет много простых методов для анимации появления и исчезновения элементов и списков, но как насчёт анимации самих данных? Например:

- чисел и арифметических выражений
- отображаемых цветов
- позиции SVG-элементов
- размеров и прочих свойств элементов

Все эти параметры либо хранятся как числа, либо могут быть сконвертированы в числа. Это значит, что мы можем использовать сторонние библиотеки для анимации, в сочетании с компонентными и реактивными системами Vue.

## Анимация состояния при помощи наблюдателей

Наблюдатели позволяют нам анимировать изменения любых числовых свойств. В такой абстрактной форме это может звучать сложновато, так что давайте просто разберём пример с использованием [GreenSock](https://greensock.com/):

{% raw %}
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.2.4/gsap.min.js"></script>

<div id="animated-number-demo">
  <input v-model.number="number" type="number" step="20">
  <p>{{ animatedNumber }}</p>
</div>
```
{% endraw %}

```js
new Vue({
  el: '#animated-number-demo',
  data: {
    number: 0,
    tweenedNumber: 0
  },
  computed: {
    animatedNumber: function() {
      return this.tweenedNumber.toFixed(0);
    }
  },
  watch: {
    number: function(newValue) {
      gsap.to(this.$data, { duration: 0.5, tweenedNumber: newValue });
    }
  }
})
```

Пример на https://ru.vuejs.org/v2/guide/transitioning-state.html

Когда вы изменяете число, это изменение в элементе под полем ввода анимуется. Для демонстрации — неплохо, но как насчёт параметров, которые напрямую как числа не хранятся, например, таких, как CSS-цвета? Используя [Tween.js](https://github.com/tweenjs/tween.js) и [Color.js](https://github.com/brehaut/color-js), мы можем решить эту задачу:

{% raw %}
```html
<script src="https://cdn.jsdelivr.net/npm/tween.js@16.3.4"></script>
<script src="https://cdn.jsdelivr.net/npm/color-js@1.0.3"></script>

<div id="example-7">
  <input
    v-model="colorQuery"
    v-on:keyup.enter="updateColor"
    placeholder="Введите цвет"
  >
  <button v-on:click="updateColor">Обновить</button>
  <p>Предпросмотр:</p>
  <span
    v-bind:style="{ backgroundColor: tweenedCSSColor }"
    class="example-7-color-preview"
  ></span>
  <p>{{ tweenedCSSColor }}</p>
</div>
```

```js
var Color = net.brehaut.Color

new Vue({
  el: '#example-7',
  data: {
    colorQuery: '',
    color: {
      red: 0,
      green: 0,
      blue: 0,
      alpha: 1
    },
    tweenedColor: {}
  },
  created: function () {
    this.tweenedColor = Object.assign({}, this.color)
  },
  watch: {
    color: function () {
      function animate () {
        if (TWEEN.update()) {
          requestAnimationFrame(animate)
        }
      }

      new TWEEN.Tween(this.tweenedColor)
        .to(this.color, 750)
        .start()

     animate()
    }
  },
  computed: {
    tweenedCSSColor: function () {
      return new Color({
        red: this.tweenedColor.red,
        green: this.tweenedColor.green,
        blue: this.tweenedColor.blue,
        alpha: this.tweenedColor.alpha
      }).toCSS()
    }
  },
  methods: {
    updateColor: function () {
      this.color = new Color(this.colorQuery).toRGB()
      this.colorQuery = ''
    }
  }
})
```

```css
.example-7-color-preview {
  display: inline-block;
  width: 50px;
  height: 50px;
}
```
{% endraw %}

Пример на https://ru.vuejs.org/v2/guide/transitioning-state.html

## Динамические переходы между состояниями

Как и для анимованных компонентов, данные, на которых основываются переходы, могут обновляться в режиме реального времени, что особенно полезно для прототипирования! Даже используя простой SVG-полигон можно достичь многих интересных эффектов, труднодостижимых без небольшой игры с переменными.

Пример на https://ru.vuejs.org/v2/guide/transitioning-state.html

См. [этот пример](https://codesandbox.io/s/github/vuejs/vuejs.org/tree/master/src/v2/examples/vue-20-dynamic-state-transitions) для просмотра исходного кода.

## Представление переходов как компонентов

Управление множеством переходов может быстро переусложнить экземпляр или компонент. К счастью, многие анимации могут быть выделены в специализированные дочерние компоненты. Давайте проделаем эту операцию с анимированным числом из одного из предыдущих примеров:

{% raw %}
```html
<script src="https://cdn.jsdelivr.net/npm/tween.js@16.3.4"></script>

<div id="example-8">
  <input v-model.number="firstNumber" type="number" step="20"> +
  <input v-model.number="secondNumber" type="number" step="20"> =
  {{ result }}
  <p>
    <animated-integer v-bind:value="firstNumber"></animated-integer> +
    <animated-integer v-bind:value="secondNumber"></animated-integer> =
    <animated-integer v-bind:value="result"></animated-integer>
  </p>
</div>
```

```js
// Эта логика перехода может быть отныне повторно использована
// с любыми целыми числами, которые мы бы хотели анимировать в приложении.
// Кроме того, компоненты предоставляют удобный интерфейс для конфигурирования
// более сложных и динамичных переходов.

Vue.component('animated-integer', {
  template: '<span>{{ tweeningValue }}</span>',
  props: {
    value: {
      type: Number,
      required: true
    }
  },
  data: function () {
    return {
      tweeningValue: 0
    }
  },
  watch: {
    value: function (newValue, oldValue) {
      this.tween(oldValue, newValue)
    }
  },
  mounted: function () {
    this.tween(0, this.value)
  },
  methods: {
    tween: function (startValue, endValue) {
      var vm = this
      function animate () {
        if (TWEEN.update()) {
          requestAnimationFrame(animate)
        }
      }

      new TWEEN.Tween({ tweeningValue: startValue })
        .to({ tweeningValue: endValue }, 500)
        .onUpdate(function () {
          vm.tweeningValue = this.tweeningValue.toFixed(0)
        })
        .start()

      animate()
    }
  }
})

// В самом экземпляре Vue больше не осталось никакой логики анимаций
new Vue({
  el: '#example-8',
  data: {
    firstNumber: 20,
    secondNumber: 40
  },
  computed: {
    result: function () {
      return this.firstNumber + this.secondNumber
    }
  }
})
```
{% endraw %}

Пример на https://ru.vuejs.org/v2/guide/transitioning-state.html

В дочерних компонентах можно использовать любую комбинацию стратегий переходов, упомянутую на этой странице, наряду со [встроенной системой переходов Vue](transitions.md). Вместе они предоставляют практически безграничные возможности.

## Оживление проектов

Анимация, в одном из определений, означает оживление. К сожалению, когда дизайнеры создают новые иконки, логотипы и талисманы, результаты обычно окажутся изображениями или статичными SVG. Таким образом осьминожек в GitHub, птичка в Twitter, и многие другие логотипы напоминают живых существ, но в действительности не кажутся живыми.

Vue может помочь. Поскольку SVG это всего лишь данные, нам нужны только примеры того, как выглядят эти существа когда они радуются, думают или встревожены. Затем Vue берёт на себя всю работу по реализации переходов между этими состояниями, помогая создавать более эмоциональные приветственные страницы, показатели загрузки и уведомления.

