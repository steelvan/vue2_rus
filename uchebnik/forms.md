
# Работа с формами

[Содержание учебника](../index.html)

## Типичное использование

Можно использовать директиву `v-model` для двунаправленнй связки данных с элементами форм input, textarea и select.  
Способ обновления элемента выбирается автоматически в зависимости от типа элемента. Хотя `v-model` и выглядит как нечто волшебное, в действительности это лишь синтаксический сахар для обновления данных в элементах ввода, с некоторыми поправками для исключительных случаев.

Директива `v-model` игнорирует начальное значение атрибутов `value`, `checked` или `selected` на любых элементах форм. Данные экземпляра Vue всегда считаются источником истины. Начальное значение необходимо объявить на стороне JavaScript, внутри объекта настроек `data` компонента.

Внутренне `v-model` использует разные свойства и генерирует разные события для различных элементов ввода:
- вводилки и текстобласть используют свойство `value` и событие `input`;
- флажки и радиокнопки используют свойство `checked` и событие `change`;
- выбиралки используют свойство `value` и событие `change`.

### Вводилка

```html
<input v-model="message" placeholder="отредактируй меня">
<p>Введённое сообщение: {{ message }}</p>
```

### Текстобласть

```html
<span>Введённое многострочное сообщение:</span>
<p style="white-space: pre-line;">{{ message }}</p>
<br>
<textarea v-model="message" placeholder="введите несколько строчек"></textarea>
```

Привязка внутри textarea (<code>&lt;textarea&gt;{{text}}&lt;/textarea&gt;</code>) не будет работать. Используйте вместо неё директиву <code>v-model</code>


### Флажки

Один флажок, привязанный к булевому значению:

```html
<input type="checkbox" id="checkbox" v-model="checked">
<label for="checkbox">{{ checked }}</label>
```
Список флажков, привязанных к одному массиву:

```html
<input type="checkbox" id="ivan" value="Иван" v-model="checkedNames">
<label for="ivan">Иван</label>
<input type="checkbox" id="petr" value="Петр" v-model="checkedNames">
<label for="petr">Петр</label>
<input type="checkbox" id="tolik" value="Толик" v-model="checkedNames">
<label for="tolik">Толик</label>
<br>
<span>Отмеченные имена: {{ checkedNames }}</span>
```

```js
new Vue({
  el: '...',
  data: {
    checkedNames: []
  }
})
```

### Радиокнопки

```html
<input type="radio" id="one" value="Один" v-model="picked">
<label for="one">Один</label>
<br>
<input type="radio" id="two" value="Два" v-model="picked">
<label for="two">Два</label>
<br>
<span>Выбрано: {{ picked }}</span>
```
### Выбиралки

Выбор одного варианта из списка:

```html
<select v-model="selected">
  <option disabled value="">Выберите один из вариантов</option>
  <option>А</option>
  <option>Б</option>
  <option>В</option>
</select>
<span>Выбрано: {{ selected }}</span>
```
```js
new Vue({
  el: '...',
  data: {
    selected: ''
  }
})
```
Если начальное значение выражения `v-model` не соответствует ни одному из вариантов списка, элемент `<select>` будет отображаться в «невыбранном» состоянии. В iOS это приведёт к тому, что пользователь не сможет выбрать первый элемент, потому что iOS не сгенерирует событие `change` в этом случае. Поэтому рекомендуется предоставлять отключённый `disabled`-вариант выбора с пустым значением value, как показано в примере выше.

Выбор нескольких вариантов из списка (с привязкой к массиву):

```html
<select v-model="selected" multiple>
  <option>А</option>
  <option>Б</option>
  <option>В</option>
</select>
<br>
<span>Выбрано: {{ selected }}</span>
```
Динамическое отображение списка выбора с помощью `v-for`:

```html
<select v-model="selected">
  <option v-for="option in options" v-bind:value="option.value">
    {{ option.text }}
  </option>
</select>
<span>Выбрано: {{ selected }}</span>
```
```js
new Vue({
  el: '...',
  data: {
    selected: 'А',
    options: [
      { text: 'Один', value: 'А' },
      { text: 'Два', value: 'Б' },
      { text: 'Три', value: 'В' }
    ]
  }
})
```
## Привязка значений

Для радиокнопок и выбиралок в качестве `v-model` обычно используются статические строки, а для флажков — булевые значения:

```html
<!-- `picked` получает строковое значение "a" при клике -->
<input type="radio" v-model="picked" value="a">

<!-- `toggle` может иметь значение true или false -->
<input type="checkbox" v-model="toggle">

<!-- `selected` при выборе первого пункта становится равным строке "abc" -->
<select v-model="selected">
  <option value="abc">ABC</option>
</select>
```

Иногда необходимо связать значение с динамическим свойством экземпляра Vue. Для этого можно использовать `v-bind`. Кроме того, использование `v-bind` позволяет связывать вводилку с нестроковыми значениями.

### Флажок

```html
<input
  type="checkbox"
  v-model="toggle"
  true-value="да"
  false-value="нет"
>
```

```js
// если чекбокс выбран:
vm.toggle === 'да'
// если чекбокс сброшен:
vm.toggle === 'нет'
```

Атрибуты `true-value` и `false-value` не влияют на атрибут `value` тега input, потому, что обозреватели пропускают невыбранные флажки при отправке форм. Чтобы гарантированно отправлять одно из двух значений с формой (например, «да» или «нет») используйте радиокнопки.

### Радиокнопки

```html
<input type="radio" v-model="pick" v-bind:value="a">
```

```js
// если отмечено:
vm.pick === vm.a
```

### Выбиралки

```html
<select v-model="selected">
  <!-- встрочный объект с данными -->
  <option v-bind:value="{ number: 123 }">123</option>
</select>
```

```js
// когда выбрано:
typeof vm.selected // => 'object'
vm.selected.number // => 123
```

## Изменители

### `.lazy`

По умолчанию `v-model` синхронизирует ввод с данными по событию `input`. Можно указать изменитель `lazy`, чтобы использовать для синхронизации _после_ события `change`:

```html
<!-- синхронизируется по событию "change", а не "input" -->
<input v-model.lazy="msg">
```

### `.number`

Для автоматического приведения введённого пользователем к Number, добавьте изменитель `number`:

```html
<input v-model.number="age" type="number">
```

Зачастую это полезно, потому что даже при указанном атрибуте `type="number"` значением вводилки всегда будет строка. Если значение не удаётся разобрать с помощью `parseFloat()`, то возвращается оригинальное значение.

### `.trim`

Если необходимо, чтобы автоматически обрезались пробелы в начале и в конце строки, используйте изменитель `trim` для полей ввода, обрабатываемых через `v-model`:

```html
<input v-model.trim="msg">
```

## Использование `v-model` с компонентами

> Если вы ещё не знакомы с компонентами Vue, пока просто пропустите эту секцию

Встроенных в HTML элементов ввода не всегда достаточно. К счастью, компоненты Vue позволяют создавать собственные аналоги с полностью настраиваемым поведением. Эти элементы тоже могут работать с директивой `v-model`!

Подробнее в разделе [пользовательские элементы ввода](components.html#Использование-v-model-на-компонентах).
