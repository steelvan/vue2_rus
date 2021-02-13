
# Анимование списков и появления/исчезновения


## Общие сведения

Vue даёт возможность самым разным образом анимовать переходы при обновлении объектов DOM, а также при их добавлении и удалении. Его инструменты дают возможность:

- автоматически применять CSS-классы для анимаций и переходов
- встраивать сторонние библиотеки CSS-анимаций, такие как Animate.css
- использовать JavaScript для работы с DOM напрямую в перехватчиках переходов
- интегрировать сторонние библиотеки JavaScript-анимаций, такие как Velocity.js

На этой странице мы рассмотрим только анимации появления и исчезновения элементов, а также анимации списков. В следующем разделе вы можете найти информацию об [анимовании переходов между состояниями](transitioning-state.md).

## Анимование одиночного элемента/компонента

Vue располагает компонентом-обёрткой `transition`, позволяющим анимовать появление и исчезновение элемента или компонента в следующих случаях:

- Условная отрисовка (используя `v-if`)
- Условное отображение (используя `v-show`)
- Динамические компоненты
- Корневые элементы компонентов

Рассмотрим в действии простой пример:

```html
<div id="demo">
  <button v-on:click="show = !show">
    Переключить
  </button>
  <transition name="fade">
    <p v-if="show">привет</p>
  </transition>
</div>
```

```js
new Vue({
  el: '#demo',
  data: {
    show: true
  }
})
```

```css
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active до версии 2.1.8 */ {
  opacity: 0;
}
```

Пример на https://ru.vuejs.org/v2/guide/transitions.html

Когда элемент, завёрнутый в компонент `transition`, вставляется или удаляется, происходят следующие действия:

1. Vue автоматически узнаёт, применены ли к целевому элементу CSS-переходы или анимации. Если да, соответствующие CSS-классы будут должным образом обновлены в нужные моменты времени.

2. Если для компонента указаны [перехватчики JavaScript](#JavaScript-перехватчики), они будут вызваны в соответствующие моменты времени.

3. Если не обнаружено ни CSS-переходов и анимаций, ни JavaScript-перехватчиков, операции DOM для вставки или удаления элемента будут выполнены непосредственно в следующем анимационном кадре (важно: речь идёт об анимационном кадре обозревателя, отличном от используемой во Vue концепции `nextTick`).

### Классы переходов

Есть шесть классов, применяющихся в анимациях появления и исчезновения элементов:

1. `v-enter`: Означает начало анимации появления элемента. Этот класс добавляется перед вставкой элемента, а удаляется в следующем кадре после вставки.

2. `v-enter-active`: Означает, что анимация появления элемента активна. Этот класс остаётся в течение всей анимации. Он добавляется перед вставкой элемента, а удаляется как только переход или анимация прекратились. Используйте его для установки длительности, задержки и плавности (easing curve) анимации появления.

3. `v-enter-to`: **Доступно только в версиях 2.1.8+.** Означает, что анимация появления элемента завершается. Класс добавляется в следующем кадре после вставки элемента (тогда же, когда будет удалён `v-enter` ), удаляется после завершения перехода или анимации.

4. `v-leave`: Означает начало анимации исчезновения элемента. Класс добавляется как только началась анимация исчезновения, а удаляется в следующем кадре после этого.

5. `v-leave-active`: Означает, что анимация исчезновения элемента активна. Этот класс остаётся в течение всей анимации. Он добавляется как только начинается анимация исчезновения, а удаляется как только переход или анимация прекратились. Используйте его для установки длительности, задержки и плавности анимации исчезновения.

6. `v-leave-to`: **Доступно только в версиях 2.1.8+.** Означает, что анимация исчезновения элемента завершается. Класс добавляется в следующем кадре после начала анимации исчезновения (тогда же, когда будет удалён `v-leave`), а удаляется после завершения перехода или анимации.

![Диаграмма переходов](/kartinki/transition.png)

Для каждого из этих классов указывается приставка, соответствующая имени перехода. Приставка `v-` применяется по умолчанию, если элемент `<transition>` используется без указания параметра `name` . Например, для `<transition name="my-transition">` вместо класса `v-enter` будет применяться `my-transition-enter`.

`v-enter-active` и `v-leave-active` дают возможность указать различные анимационные эффекты для переходов появления и исчезновения элемента. Пример их использования будет приведён ниже.

### CSS-переходы

Одни из самых распространённых типов анимации — это CSS-переходы. Вот небольшой пример:

```html
<div id="example-1">
  <button @click="show = !show">
    Переключить отрисовку
  </button>
  <transition name="slide-fade">
    <p v-if="show">привет</p>
  </transition>
</div>
```

```js
new Vue({
  el: '#example-1',
  data: {
    show: true
  }
})
```

```css
/* Анимации появления и исчезновения могут иметь */
/* различные продолжительности и динамику.       */
.slide-fade-enter-active {
  transition: all .3s ease;
}
.slide-fade-leave-active {
  transition: all .8s cubic-bezier(1.0, 0.5, 0.8, 1.0);
}
.slide-fade-enter, .slide-fade-leave-to
/* .slide-fade-leave-active до версии 2.1.8 */ {
  transform: translateX(10px);
  opacity: 0;
}
```

Пример на https://ru.vuejs.org/v2/guide/transitions.html

### CSS-анимации

CSS-анимации применяются таким же образом, как и CSS-переходы, с одним отличием: `v-enter` удаляется не сразу же после вставки элемента, а только при наступлении события `animationend`.

В этом примере обозревательные css-приставки опущены для краткости:

```html
<div id="example-2">
  <button @click="show = !show">Переключить отображение</button>
  <transition name="bounce">
    <p v-if="show">Lorem ipsum dolor sit amet, consectetur adipiscing elit. Mauris facilisis enim libero, at lacinia diam fermentum id. Pellentesque habitant morbi tristique senectus et netus.</p>
  </transition>
</div>
```

```js
new Vue({
  el: '#example-2',
  data: {
    show: true
  }
})
```

```css
.bounce-enter-active {
  animation: bounce-in .5s;
}
.bounce-leave-active {
  animation: bounce-in .5s reverse;
}
@keyframes bounce-in {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.5);
  }
  100% {
    transform: scale(1);
  }
}
```

Пример на https://ru.vuejs.org/v2/guide/transitions.html

### Пользовательские классы переходов

Вы также можете использовать пользовательские классы переходов, указывая следующие атрибуты:

- `enter-class`
- `enter-active-class`
- `enter-to-class` (в версии 2.1.8+)
- `leave-class`
- `leave-active-class`
- `leave-to-class` (в версии 2.1.8+)

Таким образом стандартные названия классов будут переопределены, что может быть особенно полезно для комбинирования системы анимованных переходов Vue с возможностями сторонних библиотек CSS-анимаций, таких как [Animate.css](https://daneden.github.io/animate.css/).

Вот пример:

```html
<link href="https://cdn.jsdelivr.net/npm/animate.css@3.5.1" rel="stylesheet" type="text/css">

<div id="example-3">
  <button @click="show = !show">
    Переключить отрисовку
  </button>
  <transition
    name="custom-classes-transition"
    enter-active-class="animated tada"
    leave-active-class="animated bounceOutRight"
  >
    <p v-if="show">привет</p>
  </transition>
</div>
```

```js
new Vue({
  el: '#example-3',
  data: {
    show: true
  }
})
```

Пример на https://ru.vuejs.org/v2/guide/transitions.html

### Совместное использование переходов и анимаций

Для того чтобы Vue знал о завершении анимации, ему требуется установка подписчиков на события. В зависимости от используемых правил CSS, таким событием будет либо `transitionend`, либо `animationend`. Если вы используете только один из подходов, Vue определит правильный тип автоматически.

Однако, иногда может потребоваться использование обоих подходов на одном элементе. Например, CSS-анимация под управлением Vue при появлении или исчезновении элемента может соседствовать с эффектом CSS-перехода при наведении курсора мыши на элемент. В таких случаях потребуется явное указание типа события, на которое должен будет ориентироваться Vue. Атрибут `type` может получить значение `animation` или `transition`.

### Указание длительности перехода

> Добавлено в версии 2.2.0+

В большинстве случаев Vue может автоматически определить когда переход завершён. По умолчанию Vue дожидается первого события `transitionend` или `animationend` на корневом элементе. Тем не менее, это не всегда может быть нужным — например, мы можем иметь хореографическую последовательность переходов, где некоторые внутренние элементы могут иметь задержку перед переходом или более длинный по времени переход, чем у корневого элемента.

В таких случаях вы можете явно указать продолжительность перехода (в миллисекундах) с помощью опции `duration` на компоненте `<transition>`:

```html
<transition :duration="1000">...</transition>
```

Можно также указать отдельные значения продолжительностей начала и окончания перехода:

```html
<transition :duration="{ enter: 500, leave: 800 }">...</transition>
```

### JavaScript-перехватчики

Можно также указывать JavaScript-перехватчики:

```html
<transition
  v-on:before-enter="beforeEnter"
  v-on:enter="enter"
  v-on:after-enter="afterEnter"
  v-on:enter-cancelled="enterCancelled"

  v-on:before-leave="beforeLeave"
  v-on:leave="leave"
  v-on:after-leave="afterLeave"
  v-on:leave-cancelled="leaveCancelled"
>
  <!-- ... -->
</transition>
```

```js
// ...
methods: {
  // --------
  // ПОЯВЛЕНИЕ
  // --------

  beforeEnter: function (el) {
    // ...
  },
  // обратновызов done не обязательно использовать, если
  // анимация или переход также определены в CSS
  enter: function (el, done) {
    // ...
    done()
  },
  afterEnter: function (el) {
    // ...
  },
  enterCancelled: function (el) {
    // ...
  },

  // --------
  // ИСЧЕЗНОВЕНИЕ
  // --------

  beforeLeave: function (el) {
    // ...
  },
  // обратновызов done не обязательно использовать, если
  // анимация или переход также определены в CSS
  leave: function (el, done) {
    // ...
    done()
  },
  afterLeave: function (el) {
    // ...
  },
  // leaveCancelled доступна только для v-show
  leaveCancelled: function (el) {
    // ...
  }
}
```

Эти хуки могут применяться как самостоятельно, так и в сочетании с CSS-переходами и анимациями.

Если ваш переход основан только на JavaScript, **обязательно вызывайте обратновызовы `done` в перехватчиках `enter` и `leave`**. Если вы этого не сделаете, перехватчики будут вызваны синхронно, и переход закончится раньше, чем отработает ваш код.

Лучше явным образом указывать `v-bind:css="false"` для переходов, основанных только на JavaScript. Это позволит Vue не тратить время на определение параметров CSS. Кроме того, это убережёт нас от случайного взаимовлияния CSS-правил и JS-перехода.

Теперь давайте разберём пример. Возьмём простой JavaScript-переход, использующий Velocity.js:

```html
<!--
Velocity работает примерно так же, как и jQuery.animate,
и весьма удобен для создания JavaScript-анимаций
-->
<script src="https://cdnjs.cloudflare.com/ajax/libs/velocity/1.2.3/velocity.min.js"></script>

<div id="example-4">
  <button @click="show = !show">
    Переключить
  </button>
  <transition
    v-on:before-enter="beforeEnter"
    v-on:enter="enter"
    v-on:leave="leave"
    v-bind:css="false"
  >
    <p v-if="show">
      Демо
    </p>
  </transition>
</div>
```

```js
new Vue({
  el: '#example-4',
  data: {
    show: false
  },
  methods: {
    beforeEnter: function (el) {
      el.style.opacity = 0
      el.style.transformOrigin = 'left'
    },
    enter: function (el, done) {
      Velocity(el, { opacity: 1, fontSize: '1.4em' }, { duration: 300 })
      Velocity(el, { fontSize: '1em' }, { complete: done })
    },
    leave: function (el, done) {
      Velocity(el, { translateX: '15px', rotateZ: '50deg' }, { duration: 600 })
      Velocity(el, { rotateZ: '100deg' }, { loop: 2 })
      Velocity(el, {
        rotateZ: '45deg',
        translateY: '30px',
        translateX: '30px',
        opacity: 0
      }, { complete: done })
    }
  }
})
```

Пример на https://ru.vuejs.org/v2/guide/transitions.html

## Переходы при первичной отрисовке

Если вы хотите, чтобы пользователь увидел анимацию перехода и при изначальной отрисовке, добавьте атрибут `appear`:

```html
<transition appear>
  <!-- ... -->
</transition>
```

По умолчанию будут задействованы переходы, указанные для появления и исчезновения. Можно, впрочем, указать и отдельные:

```html
<transition
  appear
  appear-class="custom-appear-class"
  appear-to-class="custom-appear-to-class" (в версии 2.1.8+)
  appear-active-class="custom-appear-active-class"
>
  <!-- ... -->
</transition>
```

То же справедливо и для перехватчиков:

```html
<transition
  appear
  v-on:before-appear="customBeforeAppearHook"
  v-on:appear="customAppearHook"
  v-on:after-appear="customAfterAppearHook"
  v-on:appear-cancelled="customAppearCancelledHook"
>
  <!-- ... -->
</transition>
```

В примере выше, либо атрибут `appear`, либо перехватчик `v-on:appear` вызовут появление перехода.

## Переходы между элементами

Подробно [переходы между компонентами](#Переходы-между-компонентами) мы обсудим позднее, а сейчас поговорим о переходах с помощью `v-if`/`v-else`. Один из наиболее частых случаев — переход от списка к сообщению, что список пуст:

```html
<transition>
  <table v-if="items.length > 0">
    <!-- ... -->
  </table>
  <p v-else>Жаль, но ничего не найдено.</p>
</transition>
```

Это сработает, но следует знать о нескольких скользких моментах:

При переключении между элементами, **использующими одноимённые теги**, нужно указать Vue, что это различные элементы, установив уникальные значения атрибута `key`. В противном случае компилятор Vue из соображений эффективности только поменяет содержимое элемента. Несмотря на отсутствие технической необходимости, **считается хорошим тоном всегда указывать атрибут `key` у множественных тегов в компоненте `<transition>`.**

Например:

```html
<transition>
  <button v-if="isEditing" key="save">
    Сохранить
  </button>
  <button v-else key="edit">
    Редактировать
  </button>
</transition>
```

В этом случае вы можете использовать атрибут `key` для перехода между разными состояниями элемента. Вместо использования `v-if` и `v-else`, можно переписать пример выше вот так:

{% raw %}
```html
<transition>
  <button v-bind:key="isEditing">
    {{ isEditing ? 'Сохранить' : 'Редактировать' }}
  </button>
</transition>
```
{% endraw %}

В действительности можно не ограничиваться двумя вариантами, и организовать переходы между любым количеством элементов, либо многократно используя `v-if`, либо привязывая единственный элемент к динамическому свойству. Например:

```html
<transition>
  <button v-if="docState === 'saved'" key="saved">
    Редактировать
  </button>
  <button v-if="docState === 'edited'" key="edited">
    Сохранить
  </button>
  <button v-if="docState === 'editing'" key="editing">
    Отмена
  </button>
</transition>
```

Что можно также записать в таком виде:

{% raw %}
```html
<transition>
  <button v-bind:key="docState">
    {{ buttonMessage }}
  </button>
</transition>
```
{% endraw %}

```js
// ...
computed: {
  buttonMessage: function () {
    switch (this.docState) {
      case 'saved': return 'Редактировать'
      case 'edited': return 'Сохранить'
      case 'editing': return 'Отмена'
    }
  }
}
```

### Режимы переходов

Однако сохраняется одна проблема. Попробуйте щелкнуть на кнопке ниже:

Пример на https://ru.vuejs.org/v2/guide/transitions.html

Во время перехода от кнопки «on» к кнопке «off» одновременно отображаются обе кнопки: одна — исчезая, другая — появляясь. Так `<transition>` ведёт себя по умолчанию.

Иногда это поведение подходит, например если оба элемента абсолютно спозиционированы в одном и том же месте:

Пример на https://ru.vuejs.org/v2/guide/transitions.html

Таким образом можно также сымитировать анимацию слайдера:

Пример на https://ru.vuejs.org/v2/guide/transitions.html

Тем не менее, одновременное сокрытие и появление элементов — это не всегда то, чего хочется. Поэтому Vue предоставляет альтернативные **режимы перехода**:

- `in-out`: Сначала появляется новый элемент, и только после этого исчезает старый.

- `out-in`: Сначала исчезает старый элемент, и только после этого появляется новый.

Давайте теперь изменим переход для наших on/off кнопок, используя `out-in`:

```html
<transition name="fade" mode="out-in">
  <!-- ... кнопки ... -->
</transition>
```

Пример на https://ru.vuejs.org/v2/guide/transitions.html

Добавив всего лишь один атрибут, мы исправили исходную анимацию перехода, не прибегая к редактированию стилей.

Режим `in-out` применяется не столь часто, но для достижения некоторых эффектов и он может быть полезен. Давайте попробуем совместить его с ранее рассмотренной анимацией слайдера:

Пример на https://ru.vuejs.org/v2/guide/transitions.html

Круто, правда?

## Переходы между компонентами

Переходы между компонентами — ещё проще, нам даже не нужен атрибут `key`. Всё, что нужно — завернуть [динамический компонент](components.md#Динамическое-переключение-компонентов) в `<transition>`:

```html
<transition name="component-fade" mode="out-in">
  <component v-bind:is="view"></component>
</transition>
```

```js
new Vue({
  el: '#transition-components-demo',
  data: {
    view: 'v-a'
  },
  components: {
    'v-a': {
      template: '<div>Компонент А</div>'
    },
    'v-b': {
      template: '<div>Компонент Б</div>'
    }
  }
})
```

```css
.component-fade-enter-active, .component-fade-leave-active {
  transition: opacity .3s ease;
}
.component-fade-enter, .component-fade-leave-to
/* .component-fade-leave-active до версии 2.1.8 */ {
  opacity: 0;
}
```

Пример на https://ru.vuejs.org/v2/guide/transitions.html

## Переходы в списках

Мы уже разобрались с переходами для:

- Одиночных элементов
- Множеств элементов, в случаях когда единовременно не может отображаться более одного элемента из множества

А как насчёт ситуации, когда у нас есть целый список элементов, который мы бы хотели отображать одновременно, например директивой `v-for`? В этом случае, мы используем компонент `<transition-group>`. Прежде чем мы перейдём к примеру, важно отметить несколько важных моментов, касающихся этого компонента:

- В отличие от `<transition>`, он создаёт реальный элемент. По умолчанию это `<span>`, но можно изменить на любой другой, указав атрибут `tag`.
- [Режимы переходов](#Режимы-переходов) недоступны, так как мы больше не переключаемся туда-сюда между взаимоисключающими элементами.
- У каждого элемента внутри `<transition-group>` **всегда обязательно** должно быть уникальное значение атрибута `key`.
- CSS-классы переходов будут применяться к внутренним элементам, а не к самой группе/контейнеру.

### Анимация вставки и удаления элементов списка

Давайте теперь разберём несложный пример анимации перехода вставки и удаления, использующий те же CSS-классы, что мы рассматривали ранее:

{% raw %}
```html
<div id="list-demo">
  <button v-on:click="add">Добавить</button>
  <button v-on:click="remove">Удалить</button>
  <transition-group name="list" tag="p">
    <span v-for="item in items" v-bind:key="item" class="list-item">
      {{ item }}
    </span>
  </transition-group>
</div>
```
{% endraw %}

```js
new Vue({
  el: '#list-demo',
  data: {
    items: [1,2,3,4,5,6,7,8,9],
    nextNum: 10
  },
  methods: {
    randomIndex: function () {
      return Math.floor(Math.random() * this.items.length)
    },
    add: function () {
      this.items.splice(this.randomIndex(), 0, this.nextNum++)
    },
    remove: function () {
      this.items.splice(this.randomIndex(), 1)
    },
  }
})
```

```css
.list-item {
  display: inline-block;
  margin-right: 10px;
}
.list-enter-active, .list-leave-active {
  transition: all 1s;
}
.list-enter, .list-leave-to /* .list-leave-active до версии 2.1.8 */ {
  opacity: 0;
  transform: translateY(30px);
}
```

Пример на https://ru.vuejs.org/v2/guide/transitions.html

С этим примером есть одна проблема. При добавлении или удалении элемента, окружающие его элементы резко перемещаются, вместо того чтобы плавно перелететь на новые места. Мы исправим это позднее.

### Анимация перемещения элементов списка

У компонента `<transition-group>` припасён ещё один козырь в рукаве. Он способен анимировать не только появление и удаление элементов, но также и их перемещение. Происходит это путём добавления **класса `v-move`**, который указывается при изменении позиции элементов. Как и с другими классами, префикс устанавливается в соответствии с указанным значением атрибута `name`. Аналогично, можно указать класс вручную в атрибуте `move-class`.

Этим классом удобнее всего указывать продолжительность и отсечки перехода:

{% raw %}
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.14.1/lodash.min.js"></script>

<div id="flip-list-demo" class="demo">
  <button v-on:click="shuffle">Перемешать</button>
  <transition-group name="flip-list" tag="ul">
    <li v-for="item in items" v-bind:key="item">
      {{ item }}
    </li>
  </transition-group>
</div>
```
{% endraw %}

```js
new Vue({
  el: '#flip-list-demo',
  data: {
    items: [1,2,3,4,5,6,7,8,9]
  },
  methods: {
    shuffle: function () {
      this.items = _.shuffle(this.items)
    }
  }
})
```

```css
.flip-list-move {
  transition: transform 1s;
}
```

Пример на https://ru.vuejs.org/v2/guide/transitions.html

Хотя это и может выглядеть как магия, «под капотом» Vue использует довольно простую анимационную технику под названием [FLIP](https://aerotwist.com/blog/flip-your-animations/), которая позволяет плавно перевести элементы с их старых позиций на новые, используя css-превращения.

Теперь мы можем совместить эту технику с нашим предыдущим примером, чтобы анимировать все возможные изменения нашего списка!

{% raw %}
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.14.1/lodash.min.js"></script>

<div id="list-complete-demo" class="demo">
  <button v-on:click="shuffle">Перемешать</button>
  <button v-on:click="add">Добавить</button>
  <button v-on:click="remove">Удалить</button>
  <transition-group name="list-complete" tag="p">
    <span
      v-for="item in items"
      v-bind:key="item"
      class="list-complete-item"
    >
      {{ item }}
    </span>
  </transition-group>
</div>
```
{% endraw %}
```js
new Vue({
  el: '#list-complete-demo',
  data: {
    items: [1,2,3,4,5,6,7,8,9],
    nextNum: 10
  },
  methods: {
    randomIndex: function () {
      return Math.floor(Math.random() * this.items.length)
    },
    add: function () {
      this.items.splice(this.randomIndex(), 0, this.nextNum++)
    },
    remove: function () {
      this.items.splice(this.randomIndex(), 1)
    },
    shuffle: function () {
      this.items = _.shuffle(this.items)
    }
  }
})
```

```css
.list-complete-item {
  transition: all 1s;
  display: inline-block;
  margin-right: 10px;
}
.list-complete-enter, .list-complete-leave-to
/* .list-complete-leave-active до версии 2.1.8 */ {
  opacity: 0;
  transform: translateY(30px);
}
.list-complete-leave-active {
  position: absolute;
}
```

Пример на https://ru.vuejs.org/v2/guide/transitions.html

Важно знать, что FLIP-анимации не работают с элементами, для которых установлен режим позиционирования `display: inline`. Вместо него используйте режим `display: inline-block` или flex-контейнер.

FLIP-анимации не ограничены одной осью. Многомерные массивы могут быть анимированы [столь же просто](https://codesandbox.io/s/github/vuejs/vuejs.org/tree/master/src/v2/examples/vue-20-list-move-transitions):

Пример на https://ru.vuejs.org/v2/guide/transitions.html

### Упругая анимация элементов списка

Настраивая JavaScript-переходы через data-атрибуты возможно также организовать упругую анимацию списка:

{% raw %}
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/velocity/1.2.3/velocity.min.js"></script>

<div id="staggered-list-demo">
  <input v-model="query">
  <transition-group
    name="staggered-fade"
    tag="ul"
    v-bind:css="false"
    v-on:before-enter="beforeEnter"
    v-on:enter="enter"
    v-on:leave="leave"
  >
    <li
      v-for="(item, index) in computedList"
      v-bind:key="item.msg"
      v-bind:data-index="index"
    >{{ item.msg }}</li>
  </transition-group>
</div>
```
{% endraw %}

```js
new Vue({
  el: '#staggered-list-demo',
  data: {
    query: '',
    list: [
      { msg: 'Брюс Ли' },
      { msg: 'Джеки Чан' },
      { msg: 'Чак Норрис' },
      { msg: 'Джет Ли' },
      { msg: 'Кунг Фьюри' }
    ]
  },
  computed: {
    computedList: function () {
      var vm = this
      return this.list.filter(function (item) {
        return item.msg.toLowerCase().indexOf(vm.query.toLowerCase()) !== -1
      })
    }
  },
  methods: {
    beforeEnter: function (el) {
      el.style.opacity = 0
      el.style.height = 0
    },
    enter: function (el, done) {
      var delay = el.dataset.index * 150
      setTimeout(function () {
        Velocity(
          el,
          { opacity: 1, height: '1.6em' },
          { complete: done }
        )
      }, delay)
    },
    leave: function (el, done) {
      var delay = el.dataset.index * 150
      setTimeout(function () {
        Velocity(
          el,
          { opacity: 0, height: 0 },
          { complete: done }
        )
      }, delay)
    }
  }
})
```

Пример на https://ru.vuejs.org/v2/guide/transitions.html

## Повторное использование анимованных переходов

Анимация переходов может быть переиспользована благодаря компонентной системе Vue. Всё, что необходимо сделать — это поместить компонент `<transition>` или `<transition-group>` в корне компонента, а затем передать в этот компонент потомков.

Пример с использованием компонента со строковым шаблоном:

```js
Vue.component('my-special-transition', {
  template: '\
    <transition\
      name="very-special-transition"\
      mode="out-in"\
      v-on:before-enter="beforeEnter"\
      v-on:after-enter="afterEnter"\
    >\
      <slot></slot>\
    </transition>\
  ',
  methods: {
    beforeEnter: function (el) {
      // ...
    },
    afterEnter: function (el) {
      // ...
    }
  }
})
```

Особенно хорошо для этих целей подходят [функциональные компоненты](render-function.md#Функциональные-компоненты):

```js
Vue.component('my-special-transition', {
  functional: true,
  render: function (createElement, context) {
    var data = {
      props: {
        name: 'very-special-transition',
        mode: 'out-in'
      },
      on: {
        beforeEnter: function (el) {
          // ...
        },
        afterEnter: function (el) {
          // ...
        }
      }
    }
    return createElement('transition', data, context.children)
  }
})
```

## Динамические переходы

Да, даже анимационные переходы во Vue управляются данными! Простейший пример — связывание атрибута `name` с динамическим свойством.

```html
<transition v-bind:name="transitionName">
  <!-- ... -->
</transition>
```

Может пригодиться, если вы определили CSS-переходы или анимации, используя принятые во Vue соглашения об именовании классов, и просто хотите переключаться между ними.

На самом деле, любой атрибут может быть динамически связан. И речь не только об атрибутах. Поскольку перехватчики — это просто методы, у них есть доступ ко всем данным в текущем контексте, а значит и JavaScript-анимации могут зависеть от состояния компонента.

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/velocity/1.2.3/velocity.min.js"></script>

<div id="dynamic-fade-demo" class="demo">
  Fade In: <input type="range" v-model="fadeInDuration" min="0" v-bind:max="maxFadeDuration">
  Fade Out: <input type="range" v-model="fadeOutDuration" min="0" v-bind:max="maxFadeDuration">
  <transition
    v-bind:css="false"
    v-on:before-enter="beforeEnter"
    v-on:enter="enter"
    v-on:leave="leave"
  >
    <p v-if="show">привет</p>
  </transition>
  <button
    v-if="stop"
    v-on:click="stop = false; show = false"
  >Запустить анимацию</button>
  <button
    v-else
    v-on:click="stop = true"
  >Остановить анимацию</button>
</div>
```

```js
new Vue({
  el: '#dynamic-fade-demo',
  data: {
    show: true,
    fadeInDuration: 1000,
    fadeOutDuration: 1000,
    maxFadeDuration: 1500,
    stop: true
  },
  mounted: function () {
    this.show = false
  },
  methods: {
    beforeEnter: function (el) {
      el.style.opacity = 0
    },
    enter: function (el, done) {
      var vm = this
      Velocity(el,
        { opacity: 1 },
        {
          duration: this.fadeInDuration,
          complete: function () {
            done()
            if (!vm.stop) vm.show = false
          }
        }
      )
    },
    leave: function (el, done) {
      var vm = this
      Velocity(el,
        { opacity: 0 },
        {
          duration: this.fadeOutDuration,
          complete: function () {
            done()
            vm.show = true
          }
        }
      )
    }
  }
})
```

Пример на https://ru.vuejs.org/v2/guide/transitions.html

Наконец, ещё больше возможностей для создания динамических переходов появляется, если создать компоненты, принимающие входные параметры, определяющие природу используемых переходов. Звучит избито, но всё равно — здесь всё ограничено только вашим воображением.
