ДЗ: Functional OOP
===

Password
---

```javascript
class Password {
    constructor(parent, open) {
        .....
    }
}

let p = new Password(document.body, true)

p.onChange = data => console.log(data)
p.onOpenChange = open => console.log(open)

p.setValue('qwerty')
console.log(p.getValue())

p.setOpen(false)
console.log(p.getOpen())

```

Напишите класс `Password`, которая будет в родительском элементе создавать поле ввода для пароля и 
кнопку/иконку/чекбокс, который будет переключать режим просмотра пароля в поле ввода.

Параметры:
- `parent` - родительский элемент
- `open` - стартовое состояние

Методы:
- `setValue`/`getValue` - задают/читают значения
- `setOpen`/`getOpen` - задают/читают открытость текста в поле ввода

Колбэки (функции обратного вызова, данные изнутри объекта):
- `onChange` - запускается по событию `oninput` в поле ввода, передает текст наружу
- `onOpenChange` - запускается по изменению состояния открытости пароля

LoginForm
---
С помощью предыдущего кода `Password` сделайте форму логина, кнопка в которой
будет активна только если и login и пароль не пусты


## LoginForm Constructor

оформите предыдущую задачу как класс. Продумайте и предусмотрите 
геттеры, сеттеры и колбэки.

## Password Verify

С помощью `Password` сделайте пару инпутов, которые проверяют введеный пароль
(в двух полях ввода) на совпадение. Кнопка должна активизироваться при
совпадающих паролях. При открытом пароле второе поле вводы должно пропадать с экрана
Таким образом:
 - Когда `Password` в скрытом режиме - появляется второй инпут (`<input type='password'>`) с паролем в скрытом режиме
 - Когда `Password` в открытом режиме - второй инпут пропадат


Form
---

```javascript
class Form {
    constructor(el, data, okCallback, cancelCallback) {
        let formBody = document.createElement('div')
        let okButton = document.createElement('button')
        okButton.innerHTML = 'OK'
    
        let cancelButton = document.createElement('button')
        cancelButton.innerHTML = 'Cancel'
    
        formBody.innerHTML = '<h1>тут будет форма после перервы</h1>'
        if (typeof okCallback === 'function'){
            formBody.appendChild(okButton);
            okButton.onclick = (e) => {
                console.log(this)
                this.okCallback(e)
            }
        }
    
        if (typeof cancelCallback === 'function'){
            formBody.appendChild(cancelButton);
            cancelButton.onclick = cancelCallback
        }
    
        el.appendChild(formBody)
    
    
        this.okCallback     = okCallback
        this.cancelCallback = cancelCallback
    
        this.data           = data
        this.validators     = {}
    }
}


let form = new Form(formContainer, {
    name: 'Anakin',
    surname: 'Skywalker',
    married: true,
    birthday: new Date((new Date).getTime() - 86400000 * 30*365)
}, () => console.log('ok'),() => console.log('cancel') )
form.okCallback = () => console.log('ok2')

form.validators.surname = (value, key, data, input) => value.length > 2 && 
                                                     value[0].toUpperCase() == value[0] &&
                                                    !value.includes(' ') ? true : 'Wrong name'
console.log(form)

```

Используя заготовку выше, реализуйте:

### Отображение

Проитерируйте объект `data` и создайте для каждой пары ключ-значение `input`, который будет позволять отредактировать значение.

### Sync

Синхронизируйте данные между input и объектом. Это значит, что при событии `oninput` любого поля должно происходить присвоение
соответствующего ключа объекта. Используйте замыкания и обработчик события внутри цикла, создающего `input`-ы

### Validators

Если в объекте `this.validators` есть ключ, совпадающий с ключем текущего инпута, запустите его, передав все 4 параметра.
Если результат:
    - `true` - считаем поле введенным правильно
    - **непустая строка** - считаем поле введенным неверно, выводим эту непустую строку где-то рядом с инпутом (это будет сообщение об ошибке),
        красим input в красный цвeт


### Many Inputs

Сделайте в конструкторе объект, который позволит создавать разные инпуты в зависимости от типа данных/класса объекта. Учтите, что
не всем типам данных достаточно одного input с placeholder, некоторые из них требуют дополнительной верстки:
```javascript
let inputCreators = {
    String: class String {
        constructor (key, value, oninput) {
            let input = document.createElement('input')
            input.type = 'text'
            input.placeholder = key
            input.value       = value
            input.oninput     = () => oninput(input.value)
            return input
        }
    },
    Boolean: class Boolean {
        constructor (key, value, oninput) {
            //label for с input type='checkbox' внутри
            return input
        }
    },
    Date: class Date {
        constructor(key, value, oninput) {
            //используйте datetime-local
        }
    },
    //и др. по вкусу, например textarea для длинных строк
}
```

При создании инпутов, в цикле, проверяйте тип данных с помощью `typeof`. Если это объект, достаньте имя
класса (т. е. имя конструктора) из свойства объекта `constructor.name`. В примере выше типы данных - 
`string` и `boolean`, а имя класса - `Date`. `typeof` же для экземпляра этого класса скажет `object`.
Для того, что бы `input datetime-local` понял объект даты, надо использовать строку следующего формата:
`"2020-04-16T15:16"` для `input.value`. Вы можете её создать используя методы объекта даты:
`getFullYear`, `getMonth`, `getHour` и [прочие](https://www.w3schools.com/js/js_date_methods.asp)

Когда вы добыли имя типа данных или имя класса - достаньте из объекта `inputCreators` нужную функцию
и запустите её для создания `input` и его окружающей верстки.

Переделайте все inputCreator для того, что бы они сами запускали
`oninput`, переданный в скобках и передавали верное значение, так как не всегда 
событие `oninput` и свойство `value` DOM-элемента подходит (например для `boolean` надо 
`input.oninput = () => oninput(input.checked)` а не `input.value`)

### Validators messages

Измените верстку вашей формы так, что бы возле каждого поля появился элемент для возможного текста ошибки.
Выводите там ошибку если валидатор этого поля возвращает строку (текст ошибки), а не `true`

### Mandatory

Если имя поля начинается с `*`, то:
    - поле обязательное
    - при отображении `*` должна быть красной и визуально отделена от имени поля
    - если поле пустое, то подсветить поле ввода

### Password

Если в значении того или иного поля редактируемого объекта находятся одни `*` (в любом количестве),
то данный инпут должен быть для пароля(`<input type='text'/>`), и с пустым текстом. Добавьте проверку
в метод `string` объекта `inputCreators`.

### OkButton

Должна:
    - отдавать текущий поредактированный объект первым параметром в `okCallback`
    - работать только когда:
        -   все обязательные поля не пусты
        -   всем валидаторам все понравилось
        
### CancelButton
    Должна сбрасывать поля ввода в состояние на момент создания объекта (`let form = new Form...`)
    
## Power Of The Form:

### Change Password
    Реализовать форму изменения пароля с валидацией паролей (как минимум на совпадение двух, а так же можно на длину, регистр, наличие цифр и т.п)

### Starwars
    Прикрутить к предыдущему заданию по [fetch и starwars](/javascript-async-promise-homework) редактирование любого объекта, используя `Form`.
    
### Power of closures
    Убедится в независимой работе нескольких форм одновременно на одной странице. Если не работает - чинить области видимости
    пока не заработает.
    
### Starwars `localStorage`
    По в `ok` сохранять поредактированный объект `localStorage`. При дальнейшей работе выводить не объект с бэка, а объект из `localStorage`.
    В качестве ключей используйте ссылки из **swapi**.
    
!!! (error)
    :exclamation:
    `localStorage` - это **глобальный объект** в браузере, значения которого переживают перезагрузку вкладки/браузера/компьютера
!!!

```javascript
localStorage.userName ? alert(`Your name is ${localStorage.userName}`) : localStorage.userName = prompt('What is your name?') 
```

попробуйте это и нажмите `f5`

#### Cached Promise

Оберните `fetch` или `myfetch` в функцию, которая:
    - Если данные найдены в `localStorage` - резолвится сразу, результатом промиса будет объект из `localStorage` (гляньте на `Promise.resolve`)
    - Если данные не найдены в `localStorage` - ведет себя как обычный `fetch`/`myfetch`
    
    