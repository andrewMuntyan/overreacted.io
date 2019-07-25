---
title: Полное руководство по useEffect
date: '2019-03-09'
spoiler: Эффекты - част ьпотока данных.
---

Вы уже написали несколько компонентов, используя [Хуки](https://ru.reactjs.org/docs/hooks-intro.html). Может даже небольшое приложение.
Вы по большей части довольны. Чувствуете себя вполне уверенно в обращении с API и использовали несколько трюков в процессе написания компонентов. Вы даже создали несколько [собственных Хуков](https://ru.reactjs.org/docs/hooks-custom.html), которые убрали повторяющуюся от компонента к компоненту логику (код похудел на 300 строк!). Вы показали все это своим коллегам. "Круто", - сказали они.

Но все же иногда при использовании `useEffect` не все части мозаики сходятся. И появляется раздражающее ощущение, что вы что-то упустили. Использование `useEffect` похоже на использование методов жизненного цикла компонента. Но похоже ли? И вы ловите себя на мысли, что некоторые вопросы остались без ответа. Вопросы типа таких:

* 🤔 Как с помощью `useEffect` повторить логику `componentDidMount`?
* 🤔 Как правильно отправлять запросы на получение данных внутри `useEffect`? Что такое `[]`?
* 🤔 Нужно ли указывать функции в качестве зависимостей Эффекта?
* 🤔 Почему иногда, при отправке запроса на получение данных внутри Эффекта, получается так, что Эффект вызывается бесконечное количество раз?
* 🤔 Почему иногда внутри своего Эффекта я вижу старые значения `state` и `props`?

Эти вопросы также приводили меня легкое замешательство, когда я только начал использовать Хуки. Даже на момент написания первых версий документации у меня все еще не было твердого понимания некоторых тонкостей.
С тех пор у меня было несколько "ага!" моментов, которыми я хочу с вами поделиться. **Это глубокое погружение сделает ответы на эти вопросы для вас очевидными.**

Чтобы *увидеть* ответы, нам нужно сделать шаг назад. Целью этой статьи не является дать вам список рецептов. Цель - помочь вам действительно "въехать" в `useEffect`. Здесь нечему будет учиться. На самом деле, мы проведем большую часть времени *раз*учиваясь.

**Только после того, как я перестал смотреть на `useEffect` Хук через призму известных мне методов жизненного цикла компонента, картинка в голове сложилась окончательно**

>“Забудь все то, чему тебя учили.” — Yoda

![Йода принюхивается к запаху воздуха. Подпись: “Я чую бекон.”](./yoda.jpg)

---

**Эта статья предполагает наличие у вас навыков использования [`useEffect`](https://ru.reactjs.org/docs/hooks-effect.html) API.**

**И еще она *очень* большая. Она как мини-книга. Я предпочитаю именно такой формат. Но я написал TLDR (To long did not read - много текста, не осилил) чуть ниже на тот случай, если вы торопитесь или вам не особо интересны подробности.**

**Если такой подробный формат вам не подходит, можете подождать пока объяснения, приведенные в статье, всплывут где-то в другом месте. Как было с React в 2013, когда он только появился. Людям нужно некоторое время, чтобы осознать ментальную модель новых подходов и полностью овладеть ими.**

---

## TLDR

Вот краткое содержание материала для тех, кто не настроен прочесть его полностью. Если какие-то его части не понятны во время прочтения этого краткого содержания, вы всегда можете проскроллить ниже и найти подробное объяснение.

Если же планируете осилить статью целиком, можете смело пропускать краткое содержание.

**🤔 Вопрос: как мне заменить `componentDidMount` на `useEffect`?**

Хотя вы и можете использовать `useEffect(fn, [])`, это будет не совсем одним и тем же. Отличие заключается в том, что `useEffect(fn, [])` будет *захватывать* `props` и `state`. Таким образом даже внутри `fn` - калбека, переданного в `useEffect(fn, [])`, вы будете работать с начальными `props` и `state`. Если вы хотите работать с чем-то “свежим”, вы можете переместить это в ref (имеется в виду [вот это](https://ru.reactjs.org/docs/refs-and-the-dom.html) или [useRef() хук](https://ru.reactjs.org/docs/hooks-reference.html#useref). Но обычно существует более простой вариант структурирования кода таким образом, чтобы вам не пришлось этого делать. Имейте в виду, что ментальная модель Эффектов отличается от модели `componentDidMount` и других методов жизненного цикла компонента и попытки найти их точные эквиваленты скорее больше вас запутают, чем помогут. Чтобы добиться результата, вам нужно "мыслить Эффектами" и осознать, что ментальная модель Эффектов скорее ближе к реализации синхронизации, чем к реагированию на события жизненного цикла компонента.

**🤔 Вопрос:  Как правильно отправлять запросы на получение данных внутри `useEffect`? Что такое `[]`?**

[Эта статья](https://www.robinwieruch.de/react-hooks-fetch-data/) - хороший пример получения данных с помощью `useEffect`. Дочитайте ее до конца! Она не так велика, как данный материал. `[]` означает, что внутри Эффекта не используются значения, которые принимают участие в потоке данных React и именно по этой причине безопасно применить Эффект *только один раз*. Часто ошибки возникают в тех случаях, когда значения *действительно* используются внутри Эффекта. Вам прийдется овладеть несколькими стратегиями (в первую очередь это `useReducer` и `useCallback`), которые позволят *избавиться от необходимости* использовать зависимости вместо того, чтобы получать ошибки в случае, если вы забыли передать зависимости в `useEffect(fn, deps[])`.

**🤔 Вопрос: Нужно ли мне указывать функции в качестве зависимостей Эффекта или нет?**

Функции, которые не используют `props` или `state`, рекомендуется выносить *наружу* из компонента. Те же, которые *используются только Эффектом*, переносить внутрь самого Эффекта. Если после этого ваш Эффект все еще использует методы, которые используются для рендера компонента (включая функции, переданные в качестве `props`), оберните их в `useCallback` и повторите процесс. Почему это важно? Функции могут "видеть" значения из `props` и `state`, т.е. они участвуют в потоке данных React. В нашем FAQ вы можете найти [более детальное объяснение](https://ru.reactjs.org/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies)

**🤔 Вопрос: Почему при использовании Эффектов я иногда получаю бесконечный цикл отправки запросов на получение данных?**

Это может происходить в том случае, если вы отправляете запрос на получение данных внутри Эффекта, которому в качестве второго аргумента *не были переданы зависимости*  `useEffect(fn)`. Если не передать зависимости в Эффект, он будет запускаться **при каждом рендере** компонента - значит изменение стейта компонента будет снова вызывать функцию-Эффект. Бесконечный цикл может появляться еще и в том случае, если вы в качестве зависимости указали значение, которое *всегда* изменяется. Вы можете определить какая именно из переданных зависимостей приводит к подобному поведению, убирая их одну за одной. Тем не менее удаление таких зависимостей (или просто слепое указывание `[]` в качестве зависимостей) - это обычно неверное решение проблемы. В данном случае вы скорее должны устранить источник возникновения проблемы, чем пытаться бороться с ее последствиями. Например функции могут быть источником этой проблемы и помещение их внутрь эффекта, вынесение их из компонента или оборачивание их в `useCallback` устранит проблему. Во избежание постоянного пересоздания новых инстансов объектов можно использовать  `useMemo`.

**🤔 Вопрос: Почему иногда я получаю "старые" `props` или `state` внутри функции-Эффекта?**

Эффекты всегда "видят" `props` и `state` из того рендера, в котором они были определены. [[Такое поведение помогает избежать ошибок](/how-are-function-components-different-from-classes/), но в некоторых ситуациях может казаться раздражающим. В таких случаях вы можете явно поддерживать значения в мутируемом ref (подробнее эта техника описана в конце статьи, приведенной по ссылке). Если вы видите значения  `props` и `state` из "старого" рендера внутри эффекта, хотя никак не ожидали их там увидеть, то, скорее всего, вы забыли передать в хук  `useEffect(fn, deps[])` какие-то зависимости. Попробуйте использовать [правило линтера](https://github.com/facebook/react/issues/14920) в качестве тренировки вашей способности видеть такие зависимости. Пара дней и это станет вашим вторым Я. Ознакомьтесь также с [этим ответом](https://reactjs.org/docs/hooks-faq.html#why-am-i-seeing-stale-props-or-state-inside-my-function) в нашем FAQ.

---

Я надеюсь это краткое содержание было полезным! Если нет, давайте разбираться дальше.

---

## Каждый цикл Рендера компонента имеет свои собственные `Props` и `State`

Прежде чем мы начнем говорить об Эффектах, нам стоит поговорить о процессе рендеринга компонента.

Представим, что у нас есть счетчик. Посмотрите внимательно на подсвеченную строку в коде.

```jsx{6}
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Вы нажали {count} раз</p>
      <button onClick={() => setCount(count + 1)}>
        Нажми на меня
      </button>
    </div>
  );
}
```

Что это значит? Может быть `count` каким то образом "отслеживает" изменения в нашем `state` и обновляется автоматически когда `state` изменяется? Это могло бы быть вашей первой догадкой в том случае, если бы вы только начали изучать React, но это *не* [точная ментальная модель](https://overreacted.io/react-as-a-ui-runtime/).

**В этом примере `count` - это просто число.** Это не магический дата байндинг, не "вотчер", не “прокси” и не что-нибудь подобное вышеперечисленному. Это старое доброе число, точно такое же, как и тут:

```jsx
const count = 42;
// ...
<p>Вы нажали {count} раз</p>
// ...
```
При первом рендере компонента переменная `count`, которую мы получаем из `useState()` равна `0`. Когда мы вызываем `setCount(1)` React вызывает наш компонент-функцию снова. На этот раз `count` будет равен `1`. И так далее:

```jsx{3,11,19}
// Во время первого рендера
function Counter() {
  const count = 0; // Возвращено из useState()
  // ...
  <p>Вы нажали {count} раз</p>
  // ...
}

// После нажатия на кнопку изменяется `state` и наш компонент-функция снова вызывается
function Counter() {
  const count = 1; // Возвращено из useState()
  // ...
  <p>Вы нажали {count} раз</p>
  // ...
}

// После еще одного нажатия на кнопку наш компонент-функция снова вызывается
function Counter() {
  const count = 2; // Возвращено из useState()
  // ...
  <p>Вы нажали {count} раз</p>
  // ...
}
```

**Каждый раз, когда мы обновляем `state`, React вызывает наш компонент-функцию. Каждый результат такого вызова-рендера "видит" свое собственное значение `counter` в `state`, которое является *константой* внутри нашей функции**

Т.е. эта строка не делает какого-либо особого дата байндинга:

```jsx
<p>Вы нажали {count} раз</p>
```

**Она просто вставляет числовое значение в результат рендера**. Это число предоставлено React'ом. Когда мы вызываем `setCount`, React снова вызывает наш компонент с новым значением `count`. Затем React обновляет DOM, чтобы отобразить последний результат рендера.

Ключевой момент здесь - переменная `count` внутри каждого конкретного вызова-рендера нашего компонента не изменяется со временем. Наш компонент вызывается снова и снова и каждый из этих вызовов "видит" свое значение `count`, которое изолировано между вызовами.

*(Для более глубокого понимания этого процесса Ознакомьтесь с вот этим моим постом [React это рантайм для UI](https://overreacted.io/react-as-a-ui-runtime/).)*

## Каждый цикл Рендера компонента имеет свои собственные Обработчики Событий

Пока все идет неплохо. Но как насчет обработчиков событий?

Посмотрите на этот пример. Он вызывает `alert(count)` с задержкой в 3 секунды: 

```jsx{4-8,16-18}
function Counter() {
  const [count, setCount] = useState(0);

  function handleAlertClick() {
    setTimeout(() => {
      alert('Вы нажали: ' + count + ' раз');
    }, 3000);
  }

  return (
    <div>
      <p>Вы нажали {count} раз</p>
      <button onClick={() => setCount(count + 1)}>
        Нажми на меня
      </button>
      <button onClick={handleAlertClick}>
        Показать Alert
      </button>
    </div>
  );
}
```

Представим, что я выполняю такие действия:

* **Увеличиваю** счетчик до 3
* **Нажимаю** “Показать Alert”
* **Успеваю увеличить** счетчик до 5 до того момента, когда отработает `setTimeout`

![Демо счетчика](./counter.gif)

Как вы думаете, что покажет `alert(count)`? Покажет 5 - значение `count` в момент вызова `alert`. Или покажет 3 - значение `count` в тот момент, когда я нажал на кнопку `Показать Alert`?

----

*Осторожно, спойлеры*

---

Давайте, [попробуйте сами!](https://codesandbox.io/s/w2wxl3yo0l)

Если увиденное не совсем понятно, представьте более практический пример: приложение-чат со значением `ID` текущего адресата в `state` и кнопка `Отправить`. [Эта статья](https://overreacted.io/how-are-function-components-different-from-classes/) подробно рассказывает о причинах, но правильный ответ - `alert` покажет 3

Вызов `alert` "захватывает" `state` в тот момент, когда я нажимаю на кнопку.

*(Существуют пути реализации и другого поведения, когда `alert` покажет 5, но, на данный момент, я буду фокусироваться на базовом примере. Когда формируешь ментальную модель чего-либо, важно отличать "путь наименьшего сопротивления" от возможного аварийного выхода.)*

---

Но как это работает?

Мы обсудили, что значение `count` является константным для каждого конкретного вызова нашей функции. Стоит подчеркнуть, что **наша функция вызывается множество раз (единожды во вермя рендера), но при каждом таком вызове значение `count` внутри функции не изменяется и равно конкретному значению, взятому из `state` этого конкретного рендера**

Такое поведение не является специфическим только для React - обычные функции ведут себя похожим образом:

```jsx{2}
function sayHi(person) {
  const name = person.name;
  setTimeout(() => {
    alert('Привет, ' + name);
  }, 3000);
}

let someone = {name: 'Dan'};
sayHi(someone);

someone = {name: 'Yuzhi'};
sayHi(someone);

someone = {name: 'Dominic'};
sayHi(someone);
```

В [этом примере](https://codesandbox.io/s/mm6ww11lk8), внешней переменной `someone` несколько раз присваивается новое значение. (Точно также, как и в React, *текущее* значение `state` компонента может изменяться.) **Тем не менее, внутри функции `sayHi` есть переменная `name`, значение которой берется из аргумента `person`. Этот аргумент меняется от вызова к вызову** Эта переменная является локально, таким образом ее значения изолированы между вызовами функции `sayHi`! В итоге, в момент срабатывания `SetTimeout` `alert` будет вызван с конкретным, "запомненным" локальным значением переменной `name`.

Это объясняет как на обработчик события "захватывает" значение `count` во время клика по кнопке. Если применить такой же принцип замещения, то каждый рендер "видит" свое личное значение `count`:

```jsx{3,15,27}
// Во время первого рендера
function Counter() {
  const count = 0; // Возвращено из useState()
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('Вы нажали: ' + count + ' раз');
    }, 3000);
  }
  // ...
}

// После нажатия на кнопку наша функция вызывается снова
function Counter() {
  const count = 1; // Возвращено из useState()
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('Вы нажали: ' + count + ' раз');
    }, 3000);
  }
  // ...
}

// После еще одного нажатия на кнопку наша функция вызывается снова
function Counter() {
  const count = 2; // Возвращено из useState()
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('Вы нажали: ' + count + ' раз');
    }, 3000);
  }
  // ...
}
```

Таким образом, каждый рендер возвращает свою "версию" `handleAlertClick`. Каждая из этих версий помнит свое собственное значение `count`:

```jsx{6,10,19,23,32,36}
// Во время первого рендера
function Counter() {
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('Вы нажали: ' + 0 + ' раз');
    }, 3000);
  }
  // ...
  <button onClick={handleAlertClick} /> // Этот экземпляр `handleAlertClick` "запомнил" значение 0
  // ...
}

// После нажатия на кнопку наша функция вызывается снова
function Counter() {
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('Вы нажали: ' + 1 + ' раз');
    }, 3000);
  }
  // ...
  <button onClick={handleAlertClick} /> // Этот экземпляр `handleAlertClick` "запомнил" значение 1
  // ...
}

// После еще одного нажатия на кнопку наша функция вызывается снова
function Counter() {
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('Вы нажали: ' + 2 + ' раз');
    }, 3000);
  }
  // ...
  <button onClick={handleAlertClick} /> // Этот экземпляр `handleAlertClick` "запомнил" значение 2
  // ...
}
```

Вот почему [в этом примере](https://codesandbox.io/s/w2wxl3yo0l) обработчик события "принадлежит" конкретному рендеру, и, после того, как вы нажали на кнопку, он продолжает использовать значение  `counter` *из этого* рендера.

**Внутри любого конкретного рендера `props` и `state` всегда остаются неизменными.** Поскольку `props` и `state` изолированы между рендерами, то и любые значения/выражения, использующие их (включая обработчики событий), тоже изолированы между рендерами. Они также «принадлежат» определенному рендеру. Таким образом, даже асинхронные функции внутри обработчика событий будут “видеть” “запомненное” значение `count`, принадлежащие тому рендеру, в котором были вызваны.

*Примечание: В приведенных выше примерах, я вставлял конкретные значения `count` непосредственно в `handleAlertClick` функции. Эта ментальная замена безопасна, поскольку `count` не может изменяться в рамках одного конкретного рендера. Эти значения объявлены как `const` типа `number`. Также безопасно мы могли бы думать и о значениях других типов, таких как объекты, но только в том случае, если мы договоримся избегать мутирования `state`. Вызывать `setSomethingToState(newObj)` с вновь созданным объектом вместо того, чтобы изменять этот объект, это нормально, поскольку `state`, принадлежащий предыдущим рендерам, остается неизменным.*

```jsx{3}
function Component() {
  // something - объект вида { key: 'value' }
  const [something, setSomethingToState] = useState({ key: 'value' });
  // следующие две строки верны с точки зрения нашей "ментальной модели".
  // Мы ничего не мутируем и в следующем рендере something это { key: 'anotherValue' }
  const newSomething = { key: 'anotherValue' }
  setSomethingToState(newSomething)
  // это мутирование `state`, о котором мы говорили выше
  something.key = 'yetAnotherValue'  
}
```

## Каждый цикл Рендера компонента имеет свои собственные Эффекты

Это должен был быть пост об Эффектах, но мы еще даже не начали о них говорить! Сейчас мы это исправим. Оказывается, об Эффектах можно сказать все то, что мы говорили выше об обработчиках событий, `props` и `state`.

Давайте вернемся к примеру из [документации] (https://ru.reactjs.org/docs/hooks-effect.html):

```jsx{4-6}
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `Вы нажали: ${count} раз`;
  });

  return (
    <div>
      <p>Вы нажали: {count} раз</p>
      <button onClick={() => setCount(count + 1)}>
        Нажми на меня
      </button>
    </div>
  );
}
```

**У меня к вам вопрос: как Эффект получает "самое свежее" значение `count` из `state`?**

Может быть, есть какай-то "дата байндинг" или "вотчинг", который в реальном времени обновляет значение `count` внутри функции-Эффекта? Может быть `count` - это мутабельная переменная, значение которой постоянно изменяется React'ом, и, таким образом, Эффект всегда "видит" последнее ее значение?

Нет.

Мы уже знаем, что `count` остается неизменным в рамках одного цикла рендера компонента. Даже обработчики событий "видят" значение `count` из того рендера, к которому эти обработчики "принадлежат". Все потому, что `count` - это переменная в их области видимости. То же верно и для функции-Эффекта!

**Это не переменная `count` изменяется внутри “неизменяемого” Эффекта. Это _сама функция-Эффект_ обновляется в каждом новом цикле рендера**

Каждая новая версия функции-Эффекта "видит" значение `count` из цикла рендера, к которому "принадлежит" эта функция-Эффект:

```jsx{5-8,17-20,29-32}
// Во время первого цикла рендера
function Counter() {
  // ...
  useEffect(
    // функция-Эффект из первого цикла рендера
    () => {
      document.title = `Вы нажали ${0} раз`;
    }
  );
  // ...
}

// После нажатия на кнопку наша функция-Компонент вызывается снова,
// Происходит еще один цикл рендера
function Counter() {
  // ...
  useEffect(
    // уже "другая/новая" функция-Эффект из второго цикла рендера
    () => {
       document.title = `Вы нажали ${1} раз`;
    }
  );
  // ...
}

// После еще одного нажатия наша функция-Компонент вызывается снова
function Counter() {
  // ...
  useEffect(
    // функция-Эффект из третьего цикла рендера
    () => {
       document.title = `Вы нажали ${2} раз`;
    }
  );
  // ..
}
```

React запоминает функцию, которую вы передали в Эффект, и вызывает ее после отправки браузеру свежей версии DOM, позволяя ему перерисовать экран.

Таким образом, даже если мы говорим о единственном концептуальном **эффекте** (обновление заголовка документа), то он представлен **разными функциями** для каждого цикла рендера. И каждая такая функция "видит" `props` и `state` из конкретного цикла рендера, того, к которому "принадлежит".

**Концептуально, вы можете думать об Эффектах, как о *части результатов рендера*.**

Строго говоря, они не являются результатами рендера (с целью обеспечения [возможности композиции Хуков](https://overreacted.io/why-do-hooks-rely-on-call-order/) без применения странного синтаксиса и ухудшения быстродействия). Но в ментальной модели, которую мы строим, функции-Эффекты *принадлежат* к конкретному циклу рендера точно также, как это происходит с обработчиками событий.

---

Чтобы убедиться, что у нас есть четкое понимание, давайте пройдемся по нашему первому рендеру:

* **React:** Дай мне UI когда `count` в `state` равен `0`.
* **Ваш компонент:**
  * Вот результаты рендера:
  `<p>Вы нажали 0 раз</p>`.
  * И еще не забудь выполнить этот Эффект, после того, как закончишь: `() => { document.title = 'Вы нажали 0 раз' }`.
* **React:** Конечно. Обновляю UI. Эй, Браузер, я кое-что добавляю в DOM.
* **Браузер:** Круто, сейчас я отрисую это на экране.
* **React:** OK, сейчас я собираюсь выполнить тот Эффект, который дал мне компонент.
  * Выполняю `() => { document.title = 'Вы нажали 0 раз' }`.

---

Теперь давайте вспомним что происходит после нажатия на кнопку:

* **Ваш компонент:** Эй React, измени значение `count` в `state` с `0` на `1`.
* **React:** Дай мне UI когда `count` в `state` равен `1`.
* **Ваш компонент:**
  * Вот результаты рендера:
  `<p>Вы нажали 1 раз</p>`.
  * И еще не забудь выполнить этот Эффект, после того, как закончишь: `() => { document.title = 'Вы нажали 1 раз' }`.
* **React:** Конечно. Обновляю UI. Эй, Браузер, я изменил DOM.
* **Браузер:** Круто, я отрисовал твои изменения на экране.
* **React:** OK, теперь я запущу Эффект, которая принадлежит к рендеру, который я только что сделал.
  * Запускаю `() => { document.title = 'You clicked 1 times' }`.

---

## Каждый цикл Рендера компонента имеет свои собственные... Да Все

**Теперь мы знаем, что Эффекты, которые запускаются после каждого рендера, концептуально являются частью того, что компонент отдает React'у. Мы знаем также, что Эффекты "видят" те `props` и `state`, которые принадлежат к конкретному циклу рендера**

Давайте проведем мысленный эксперимент. Представьте такой код:

```jsx{4-8}
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    setTimeout(() => {
      console.log(`Вы нажали ${count} раз`);
    }, 3000);
  });

  return (
    <div>
      <p>Вы нажали {count} раз</p>
      <button onClick={() => setCount(count + 1)}>
        Нажми меня
      </button>
    </div>
  );
}
```

Как будет выглядеть результат `console.log`, если я нажму на кнопку несколько раз с небольшими перерывами?

---

*Осторожно, спойлеры*

---

Вы можете подумать, что это ловушка, и что конечный результат не логичен. Но это не так! Мы увидим последовательность результатов `console.log` - каждый из которых принадлежит определенному рендеру и, таким образом, имеет свое собственное значение `count`. Можете [сами попробовать](https://codesandbox.io/s/lyx20m1ol):

![Запись экрана, на которой показано как 1, 2, 3, 4, 5 выводятся последовательно](./timeout_counter.gif)

Вы можете подумать: «Конечно это работает так! Как еще это может работать?

Ну, это отличается от того, как `this.state` работает в классах. Легко ошибиться, если подумать, что вот в такой [реализации с помощью класса](https://codesandbox.io/s/kkymzwjqz3) мы увидим тот же результат:


```jsx
  componentDidUpdate() {
    setTimeout(() => {
      console.log(`Вы нажали ${this.state.count} раз`);
    }, 3000);
  }
```

However, `this.state.count` always points at the *latest* count rather than the one belonging to a particular render. So you’ll see `5` logged each time instead:

![Screen recording of 5, 5, 5, 5, 5 logged in order](./timeout_counter_class.gif)

I think it’s ironic that Hooks rely so much on JavaScript closures, and yet it’s the class implementation that suffers from [the canonical wrong-value-in-a-timeout confusion](https://wsvincent.com/javascript-closure-settimeout-for-loop/) that’s often associated with closures. This is because the actual source of the confusion in this example is the mutation (React mutates `this.state` in classes to point to the latest state) and not closures themselves.

**Closures are great when the values you close over never change. That makes them easy to think about because you’re essentially referring to constants.** And as we discussed, props and state never change within a particular render. By the way, we can fix the class version... by [using a closure](https://codesandbox.io/s/w7vjo07055).

## Swimming Against the Tide

At this point it’s important that we call it out explicitly: **every** function inside the component render (including event handlers, effects, timeouts or API calls inside them) captures the props and state of the render call that defined it.

So these two examples are equivalent:

```jsx{4}
function Example(props) {
  useEffect(() => {
    setTimeout(() => {
      console.log(props.counter);
    }, 1000);
  });
  // ...
}
```

```jsx{2,5}
function Example(props) {
  const counter = props.counter;
  useEffect(() => {
    setTimeout(() => {
      console.log(counter);
    }, 1000);
  });
  // ...
}
```

**It doesn’t matter whether you read from props or state “early” inside of your component.** They’re not going to change! Inside the scope of a single render, props and state stay the same. (Destructuring props makes this more obvious.)

Of course, sometimes you *want* to read the latest rather than captured value inside some callback defined in an effect. The easiest way to do it is by using refs, as described in the last section of [this article](https://overreacted.io/how-are-function-components-different-from-classes/).

Be aware that when you want to read the *future* props or state from a function in a *past* render, you’re swimming against the tide. It’s not *wrong* (and in some cases necessary) but it might look less “clean” to break out of the paradigm. This is an intentional consequence because it helps highlight which code is fragile and depends on timing. In classes, it’s less obvious when this happens.

Here’s a [version of our counter example](https://codesandbox.io/s/rm7z22qnlp) that replicates the class behavior:

```jsx{3,6-7,9-10}
function Example() {
  const [count, setCount] = useState(0);
  const latestCount = useRef(count);

  useEffect(() => {
    // Set the mutable latest value
    latestCount.current = count;
    setTimeout(() => {
      // Read the mutable latest value
      console.log(`You clicked ${latestCount.current} times`);
    }, 3000);
  });
  // ...
```

![Screen recording of 5, 5, 5, 5, 5 logged in order](./timeout_counter_refs.gif)

It might seem quirky to mutate something in React. However, this is exactly how React itself reassigns `this.state` in classes. Unlike with captured props and state, you don’t have any guarantees that reading `latestCount.current` would give you the same value in any particular callback. By definition, you can mutate it any time. This is why it’s not a default, and you have to opt into that.

## So What About Cleanup?

As [the docs explain](https://reactjs.org/docs/hooks-effect.html#effects-with-cleanup), some effects might have a cleanup phase. Essentially, its purpose is to “undo” an effect for cases like subscriptions.

Consider this code:

```jsx
  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.id, handleStatusChange);
    };
  });
```

Say `props` is `{id: 10}` on the first render, and `{id: 20}` on the second render. You *might* think that something like this happens:

* React cleans up the effect for `{id: 10}`.
* React renders UI for `{id: 20}`.
* React runs the effect for `{id: 20}`.

(This is not quite the case.)

With this mental model, you might think the cleanup “sees” the old props because it runs before we re-render, and then the new effect “sees” the new props because it runs after the re-render. That’s the mental model lifted directly from the class lifecycles, and **it’s not accurate here**. Let’s see why.

React only runs the effects after [letting the browser paint](https://medium.com/@dan_abramov/this-benchmark-is-indeed-flawed-c3d6b5b6f97f). This makes your app faster as most effects don’t need to block screen updates. Effect cleanup is also delayed. **The previous effect is cleaned up _after_ the re-render with new props:**

* **React renders UI for `{id: 20}`.**
* The browser paints. We see the UI for `{id: 20}` on the screen.
* **React cleans up the effect for `{id: 10}`.**
* React runs the effect for `{id: 20}`.

You might be wondering: but how can the cleanup of the previous effect still “see” the old `{id: 10}` props if it runs *after* the props change to `{id: 20}`?

We’ve been here before... 🤔

![Deja vu (cat scene from the Matrix movie)](./deja_vu.gif)

Quoting the previous section:

>Every function inside the component render (including event handlers, effects, timeouts or API calls inside them) captures the props and state of the render call that defined it.

Now the answer is clear! The effect cleanup doesn’t read the “latest” props, whatever that means. It reads props that belong to the render it’s defined in:

```jsx{8-11}
// First render, props are {id: 10}
function Example() {
  // ...
  useEffect(
    // Effect from first render
    () => {
      ChatAPI.subscribeToFriendStatus(10, handleStatusChange);
      // Cleanup for effect from first render
      return () => {
        ChatAPI.unsubscribeFromFriendStatus(10, handleStatusChange);
      };
    }
  );
  // ...
}

// Next render, props are {id: 20}
function Example() {
  // ...
  useEffect(
    // Effect from second render
    () => {
      ChatAPI.subscribeToFriendStatus(20, handleStatusChange);
      // Cleanup for effect from second render
      return () => {
        ChatAPI.unsubscribeFromFriendStatus(20, handleStatusChange);
      };
    }
  );
  // ...
}
```

Kingdoms will rise and turn into ashes, the Sun will shed its outer layers to be a white dwarf, and the last civilization will end. But nothing will make the props “seen” by the first render effect’s cleanup anything other than `{id: 10}`.

That’s what allows React to deal with effects right after painting — and make your apps faster by default. The old props are still there if our code needs them.

## Synchronization, Not Lifecycle

One of my favorite things about React is that it unifies describing the initial render result and the updates. This [reduces the entropy](https://overreacted.io/the-bug-o-notation/) of your program.

Say my component looks like this:

```jsx
function Greeting({ name }) {
  return (
    <h1 className="Greeting">
      Hello, {name}
    </h1>
  );
}
```

It doesn’t matter if I render `<Greeting name="Dan" />` and later `<Greeting name="Yuzhi" />`, or if I just render `<Greeting name="Yuzhi" />`. In the end, we will see “Hello, Yuzhi” in both cases.

People say: “It’s all about the journey, not the destination”. With React, it’s the opposite. **It’s all about the destination, not the journey.** That’s the difference between `$.addClass` and `$.removeClass` calls in jQuery code (our “journey”) and specifying what the CSS class *should be* in React code (our “destination”).

**React synchronizes the DOM according to our current props and state.** There is no distinction between a “mount” or an “update” when rendering.

You should think of effects in a similar way. **`useEffect` lets you _synchronize_ things outside of the React tree according to our props and state.**

```jsx{2-4}
function Greeting({ name }) {
  useEffect(() => {
    document.title = 'Hello, ' + name;
  });
  return (
    <h1 className="Greeting">
      Hello, {name}
    </h1>
  );
}
```

This is subtly different from the familiar *mount/update/unmount* mental model. It is important really to internalize this. **If you’re trying to write an effect that behaves differently depending on whether the component renders for the first time or not, you’re swimming against the tide!** We’re failing at synchronizing if our result depends on the “journey” rather than the “destination”.

It shouldn’t matter whether we rendered with props A, B, and C, or if we rendered with C immediately. While there may be some temporary differences (e.g. while we’re fetching data), eventually the end result should be the same.

Still, of course running all effects on *every* render might not be efficient. (And in some cases, it would lead to infinite loops.)

So how can we fix this?

## Teaching React to Diff Your Effects

We’ve already learned that lesson with the DOM itself. Instead of touching it on every re-render, React only updates the parts of the DOM that actually change.

When you’re updating

```jsx
<h1 className="Greeting">
  Hello, Dan
</h1>
```

to

```jsx
<h1 className="Greeting">
  Hello, Yuzhi
</h1>
```

React sees two objects:

```jsx
const oldProps = {className: 'Greeting', children: 'Hello, Dan'};
const newProps = {className: 'Greeting', children: 'Hello, Yuzhi'};
```

It goes over each of their props and determine that `children` have changed and need a DOM update, but `className` did not. So it can just do:

```jsx
domNode.innerText = 'Hello, Yuzhi';
// No need to touch domNode.className
```

**Could we do something like this with effects too? It would be nice to avoid re-running them when applying the effect is unnecessary.**

For example, maybe our component re-renders because of a state change:

```jsx{11-13}
function Greeting({ name }) {
  const [counter, setCounter] = useState(0);

  useEffect(() => {
    document.title = 'Hello, ' + name;
  });

  return (
    <h1 className="Greeting">
      Hello, {name}
      <button onClick={() => setCounter(count + 1)}>
        Increment
      </button>
    </h1>
  );
}
```

But our effect doesn’t use the `counter` state. **Our effect synchronizes the `document.title` with the `name` prop, but the `name` prop is the same.** Re-assigning `document.title` on every counter change seems non-ideal.

OK, so can React just... diff effects?


```jsx
let oldEffect = () => { document.title = 'Hello, Dan'; };
let newEffect = () => { document.title = 'Hello, Dan'; };
// Can React see these functions do the same thing?
```

Not really. React can’t guess what the function does without calling it. (The source doesn’t really contain specific values, it just closes over the `name` prop.)

This is why if you want to avoid re-running effects unnecessarily, you can provide a dependency array (also known as “deps”) argument to `useEffect`:

```jsx{3}
  useEffect(() => {
    document.title = 'Hello, ' + name;
  }, [name]); // Our deps
```

**It’s like if we told React: “Hey, I know you can’t see _inside_ this function, but I promise it only uses `name` and nothing else from the render scope.”**

If each of these values is the same between the current and the previous time this effect ran, there’s nothing to synchronize so React can skip the effect:

```jsx
const oldEffect = () => { document.title = 'Hello, Dan'; };
const oldDeps = ['Dan'];

const newEffect = () => { document.title = 'Hello, Dan'; };
const newDeps = ['Dan'];

// React can't peek inside of functions, but it can compare deps.
// Since all deps are the same, it doesn’t need to run the new effect.
```

If even one of the values in the dependency array is different between renders, we know running the effect can’t be skipped. Synchronize all the things!

## Don’t Lie to React About Dependencies

Lying to React about dependencies has bad consequences. Intuitively, this makes sense, but I’ve seen pretty much everyone who tries `useEffect` with a mental model from classes try to cheat the rules. (And I did that too at first!)

```jsx
function SearchResults() {
  async function fetchData() {
    // ...
  }

  useEffect(() => {
    fetchData();
  }, []); // Is this okay? Not always -- and there's a better way to write it.

  // ...
}
```

*(The [Hooks FAQ](https://reactjs.org/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies) explains what to do instead. We'll come back to this example [below](#moving-functions-inside-effects).)*

“But I only want to run it on mount!”, you’ll say. For now, remember: if you specify deps, **_all_ values from inside your component that are used by the effect _must_ be there**. Including props, state, functions — anything in your component.

Sometimes when you do that, it causes a problem. For example, maybe you see an infinite refetching loop, or a socket is recreated too often. **The solution to that problem is _not_ to remove a dependency.** We’ll look at the solutions soon.

But before we jump to solutions, let’s understand the problem better.

## What Happens When Dependencies Lie

If deps contain every value used by the effect, React knows when to re-run it:

```jsx{3}
  useEffect(() => {
    document.title = 'Hello, ' + name;
  }, [name]);
```

![Diagram of effects replacing one another](./deps-compare-correct.gif)

*(Dependencies are different, so we re-run the effect.)*

But if we specified `[]` for this effect, the new effect function wouldn’t run:

```jsx{3}
  useEffect(() => {
    document.title = 'Hello, ' + name;
  }, []); // Wrong: name is missing in deps
```

![Diagram of effects replacing one another](./deps-compare-wrong.gif)

*(Dependencies are equal, so we skip the effect.)*

In this case the problem might seem obvious. But the intuition can fool you in other cases where a class solution “jumps out” from your memory.

For example, let’s say we’re writing a counter that increments every second. With a class, our intuition is: “Set up the interval once and destroy it once”. Here’s an [example](https://codesandbox.io/s/n5mjzjy9kl) of how we can do it. When we mentally translate this code to `useEffect`, we instinctively add `[]` to the deps. “I want it to run once”, right?

```jsx{9}
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);

  return <h1>{count}</h1>;
}
```

However, this example [only *increments* once](https://codesandbox.io/s/91n5z8jo7r). *Oops.*

If your mental model is “dependencies let me specify when I want to re-trigger the effect”, this example might give you an existential crisis. You *want* to trigger it once because it’s an interval — so why is it causing issues?

However, this makes sense if you know that dependencies are our hint to React about *everything* that the effect uses from the render scope. It uses `count` but we lied that it doesn’t with `[]`. It’s only a matter of time before this bites us!

In the first render, `count` is `0`. Therefore, `setCount(count + 1)` in the first render’s effect means `setCount(0 + 1)`. **Since we never re-run the effect because of `[]` deps, it will keep calling `setCount(0 + 1)` every second:**

```jsx{8,12,21-22}
// First render, state is 0
function Counter() {
  // ...
  useEffect(
    // Effect from first render
    () => {
      const id = setInterval(() => {
        setCount(0 + 1); // Always setCount(1)
      }, 1000);
      return () => clearInterval(id);
    },
    [] // Never re-runs
  );
  // ...
}

// Every next render, state is 1
function Counter() {
  // ...
  useEffect(
    // This effect is always ignored because
    // we lied to React about empty deps.
    () => {
      const id = setInterval(() => {
        setCount(1 + 1);
      }, 1000);
      return () => clearInterval(id);
    },
    []
  );
  // ...
}
```

We lied to React by saying our effect doesn’t depend on a value from inside our component, when in fact it does!

Our effect uses `count` — a value inside the component (but outside the effect):

```jsx{1,5}
  const count = // ...

  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);
```

Therefore, specifying `[]` as a dependency will create a bug. React will compare the dependencies, and skip updating this effect:

![Diagram of stale interval closure](./interval-wrong.gif)

*(Dependencies are equal, so we skip the effect.)*

Issues like this are difficult to think about. Therefore, I encourage you to adopt it as a hard rule to always be honest about the effect dependencies, and specify them all. (We provide a [lint rule](https://github.com/facebook/react/issues/14920) if you want to enforce this on your team.)

## Two Ways to Be Honest About Dependencies

There are two strategies to be honest about dependencies. You should generally start with the first one, and then apply the second one if needed.

**The first strategy is to fix the dependency array to include _all_ the values inside the component that are used inside the effect.** Let’s include `count` as a dep:

```jsx{3,6}
useEffect(() => {
  const id = setInterval(() => {
    setCount(count + 1);
  }, 1000);
  return () => clearInterval(id);
}, [count]);
```

This makes the dependency array correct. It may not be *ideal* but that’s the first issue we needed to fix. Now a change to `count` will re-run the effect, with each next interval referencing `count` from its render in `setCount(count + 1)`:

```jsx{8,12,24,28}
// First render, state is 0
function Counter() {
  // ...
  useEffect(
    // Effect from first render
    () => {
      const id = setInterval(() => {
        setCount(0 + 1); // setCount(count + 1)
      }, 1000);
      return () => clearInterval(id);
    },
    [0] // [count]
  );
  // ...
}

// Second render, state is 1
function Counter() {
  // ...
  useEffect(
    // Effect from second render
    () => {
      const id = setInterval(() => {
        setCount(1 + 1); // setCount(count + 1)
      }, 1000);
      return () => clearInterval(id);
    },
    [1] // [count]
  );
  // ...
}
```

That would [fix the problem](https://codesandbox.io/s/0x0mnlyq8l) but our interval would be cleared and set again whenever the `count` changes. That may be undesirable:

![Diagram of interval that re-subscribes](./interval-rightish.gif)

*(Dependencies are different, so we re-run the effect.)*

---

**The second strategy is to change our effect code so that it wouldn’t *need* a value that changes more often than we want.** We don’t want to lie about the dependencies — we just want to change our effect to have *fewer* of them.

Let’s look at a few common techniques for removing dependencies.

---

## Making Effects Self-Sufficient

We want to get rid of the `count` dependency in our effect.

```jsx{3,6}
  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(id);
  }, [count]);
```

To do this, we need to ask ourselves: **what are we using `count` for?** It seems like we only use it for the `setCount` call. In that case, we don’t actually need `count` in the scope at all. When we want to update state based on the previous state, we can use the [functional updater form](https://reactjs.org/docs/hooks-reference.html#functional-updates) of `setState`:

```jsx{3}
  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);
```

I like to think of these cases as “false dependencies”. Yes, `count` was a necessary dependency because we wrote `setCount(count + 1)` inside the effect. However, we only truly needed `count` to transform it into `count + 1` and “send it back” to React. But React *already knows* the current `count`. **All we needed to tell React is to increment the state — whatever it is right now.**

That’s exactly what `setCount(c => c + 1)` does. You can think of it as “sending an instruction” to React about how the state should change. This “updater form” also helps in other cases, like when you [batch multiple updates](/react-as-a-ui-runtime/#batching).

**Note that we actually _did the work_ to remove the dependency. We didn’t cheat. Our effect doesn’t read the `counter` value from the render scope anymore:**

![Diagram of interval that works](./interval-right.gif)

*(Dependencies are equal, so we skip the effect.)*

You can try it [here](https://codesandbox.io/s/q3181xz1pj).

Even though this effect only runs once, the interval callback that belongs to the first render is perfectly capable of sending the `c => c + 1` update instruction every time the interval fires. It doesn’t need to know the current `counter` state anymore. React already knows it.

## Functional Updates and Google Docs

Remember how we talked about synchronization being the mental model for effects? An interesting aspect of synchronization is that you often want to keep the “messages” between the systems untangled from their state. For example, editing a document in Google Docs doesn’t actually send the *whole* page to the server. That would be very inefficient. Instead, it sends a representation of what the user tried to do.

While our use case is different, a similar philosophy applies to effects. **It helps to send only the minimal necessary information from inside the effects into a component.** The updater form like `setCount(c => c + 1)` conveys strictly less information than `setCount(count + 1)` because it isn’t “tainted” by the current count. It only expresses the action (“incrementing”). Thinking in React involves [finding the minimal state](https://reactjs.org/docs/thinking-in-react.html#step-3-identify-the-minimal-but-complete-representation-of-ui-state). This is the same principle, but for updates.

Encoding the *intent* (rather than the result) is similar to how Google Docs [solves](https://medium.com/@srijancse/how-real-time-collaborative-editing-work-operational-transformation-ac4902d75682) collaborative editing. While this is stretching the analogy, functional updates serve a similar role in React. They ensure updates from multiple sources (event handlers, effect subscriptions, etc) can be correctly applied in a batch and in a predictable way.

**However, even `setCount(c => c + 1)` isn’t that great.** It looks a bit weird and it’s very limited in what it can do. For example, if we had two state variables whose values depend on each other, or if we needed to calculate the next state based on a prop, it wouldn’t help us. Luckily, `setCount(c => c + 1)` has a more powerful sister pattern. Its name is `useReducer`.

## Decoupling Updates from Actions

Let’s modify the previous example to have two state variables: `count` and `step`. Our interval will increment the count by the value of the `step` input:

```jsx{7,10}
function Counter() {
  const [count, setCount] = useState(0);
  const [step, setStep] = useState(1);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + step);
    }, 1000);
    return () => clearInterval(id);
  }, [step]);

  return (
    <>
      <h1>{count}</h1>
      <input value={step} onChange={e => setStep(Number(e.target.value))} />
    </>
  );
}
```

(Here’s a [demo](https://codesandbox.io/s/zxn70rnkx).)

Note that **we’re not cheating**. Since I started using `step` inside the effect, I added it to the dependencies. And that’s why the code runs correctly.

The current behavior in this example is that changing the `step` restarts the interval — because it’s one of the dependencies. And in many cases, that is exactly what you want! There’s nothing wrong with tearing down an effect and setting it up anew, and we shouldn’t avoid that unless we have a good reason.

However, let’s say we want the interval clock to not reset on changes to the `step`. How do we remove the `step` dependency from our effect?

**When setting a state variable depends on the current value of another state variable, you might want to try replacing them both with `useReducer`.**

When you find yourself writing `setSomething(something => ...)`, it’s a good time to consider using a reducer instead. A reducer lets you **decouple expressing the “actions” that happened in your component from how the state updates in response to them**.

Let’s trade the `step` dependency for a `dispatch` dependency in our effect:

```jsx{1,6,9}
const [state, dispatch] = useReducer(reducer, initialState);
const { count, step } = state;

useEffect(() => {
  const id = setInterval(() => {
    dispatch({ type: 'tick' }); // Instead of setCount(c => c + step);
  }, 1000);
  return () => clearInterval(id);
}, [dispatch]);
```

(See the [demo](https://codesandbox.io/s/xzr480k0np).)

You might ask me: “How is this any better?” The answer is that **React guarantees the `dispatch` function to be constant throughout the component lifetime. So the example above doesn’t ever need to resubscribe the interval.**

We solved our problem!

*(You may omit `dispatch`, `setState`, and `useRef` container values from the deps because React guarantees them to be static. But it also doesn’t hurt to specify them.)*

Instead of reading the state *inside* an effect, it dispatches an *action* that encodes the information about *what happened*. This allows our effect to stay decoupled from the `step` state. Our effect doesn’t care *how* we update the state, it just tells us about *what happened*. And the reducer centralizes the update logic:

```jsx{8,9}
const initialState = {
  count: 0,
  step: 1,
};

function reducer(state, action) {
  const { count, step } = state;
  if (action.type === 'tick') {
    return { count: count + step, step };
  } else if (action.type === 'step') {
    return { count, step: action.step };
  } else {
    throw new Error();
  }
}
```

(Here’s a [demo](https://codesandbox.io/s/xzr480k0np) if you missed it earlier).

## Why useReducer Is the Cheat Mode of Hooks

We’ve seen how to remove dependencies when an effect needs to set state based on previous state, or on another state variable. **But what if we need _props_ to calculate the next state?** For example, maybe our API is `<Counter step={1} />`. Surely, in this case we can’t avoid specifying `props.step` as a dependency?

In fact, we can! We can put *the reducer itself* inside our component to read props:

```jsx{1,6}
function Counter({ step }) {
  const [count, dispatch] = useReducer(reducer, 0);

  function reducer(state, action) {
    if (action.type === 'tick') {
      return state + step;
    } else {
      throw new Error();
    }
  }

  useEffect(() => {
    const id = setInterval(() => {
      dispatch({ type: 'tick' });
    }, 1000);
    return () => clearInterval(id);
  }, [dispatch]);

  return <h1>{count}</h1>;
}
```

This pattern disables a few optimizations so try not to use it everywhere, but you can totally access props from a reducer if you need to. (Here’s a [demo](https://codesandbox.io/s/7ypm405o8q).)

**Even in that case, `dispatch` identity is still guaranteed to be stable between re-renders.** So you may omit it from the effect deps if you want. It’s not going to cause the effect to re-run.

You may be wondering: how can this possibly work? How can the reducer “know” props when called from inside an effect that belongs to another render? The answer is that when you `dispatch`, React just remembers the action — but it will *call* your reducer during the next render. At that point the fresh props will be in scope, and you won’t be inside an effect.

**This is why I like to think of `useReducer` as the “cheat mode” of Hooks. It lets me decouple the update logic from describing what happened. This, in turn, helps me remove unnecessary dependencies from my effects and avoid re-running them more often than necessary.**

## Moving Functions Inside Effects

A common mistake is to think functions shouldn’t be dependencies. For example, this seems like it could work:

```jsx{13}
function SearchResults() {
  const [data, setData] = useState({ hits: [] });

  async function fetchData() {
    const result = await axios(
      'https://hn.algolia.com/api/v1/search?query=react',
    );
    setData(result.data);
  }

  useEffect(() => {
    fetchData();
  }, []); // Is this okay?

  // ...
```

*([This example](https://codesandbox.io/s/8j4ykjyv0) is adapted from a great article by Robin Wieruch — [check it out](https://www.robinwieruch.de/react-hooks-fetch-data/)!)*

And to be clear, this code *does* work. **But the problem with simply omitting local functions is that it gets pretty hard to tell whether we’re handling all cases as the component grows!**

Imagine our code was split like this and each function was five times larger:

```jsx
function SearchResults() {
  // Imagine this function is long
  function getFetchUrl() {
    return 'https://hn.algolia.com/api/v1/search?query=react';
  }

  // Imagine this function is also long
  async function fetchData() {
    const result = await axios(getFetchUrl());
    setData(result.data);
  }

  useEffect(() => {
    fetchData();
  }, []);

  // ...
}
```


Now let’s say we later use some state or prop in one of these functions:

```jsx{6}
function SearchResults() {
  const [query, setQuery] = useState('react');

  // Imagine this function is also long
  function getFetchUrl() {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }

  // Imagine this function is also long
  async function fetchData() {
    const result = await axios(getFetchUrl());
    setData(result.data);
  }

  useEffect(() => {
    fetchData();
  }, []);

  // ...
}
```

If we forget to update the deps of any effects that call these functions (possibly, through other functions!), our effects will fail to synchronize changes from our props and state. This doesn’t sound great.

Luckily, there is an easy solution to this problem. **If you only use some functions *inside* an effect, move them directly *into* that effect:**

```jsx{4-12}
function SearchResults() {
  // ...
  useEffect(() => {
    // We moved these functions inside!
    function getFetchUrl() {
      return 'https://hn.algolia.com/api/v1/search?query=react';
    }

    async function fetchData() {
      const result = await axios(getFetchUrl());
      setData(result.data);
    }

    fetchData();
  }, []); // ✅ Deps are OK
  // ...
}
```

([Here’s a demo](https://codesandbox.io/s/04kp3jwwql).)

So what is the benefit? We no longer have to think about the “transitive dependencies”. Our dependencies array isn’t lying anymore: **we truly _aren’t_ using anything from the outer scope of the component in our effect**.

If we later edit `getFetchUrl` to use the `query` state, we’re much more likely to notice that we’re editing it *inside* an effect — and therefore, we need to add `query` to the effect dependencies:

```jsx{6,15}
function SearchResults() {
  const [query, setQuery] = useState('react');

  useEffect(() => {
    function getFetchUrl() {
      return 'https://hn.algolia.com/api/v1/search?query=' + query;
    }

    async function fetchData() {
      const result = await axios(getFetchUrl());
      setData(result.data);
    }

    fetchData();
  }, [query]); // ✅ Deps are OK

  // ...
}
```

(Here’s a [demo](https://codesandbox.io/s/pwm32zx7z7).)

By adding this dependency, we’re not just “appeasing React”. It *makes sense* to refetch the data when the query changes. **The design of `useEffect` forces you to notice the change in our data flow and choose how our effects should synchronize it — instead of ignoring it until our product users hit a bug.**

Thanks to the `exhaustive-deps` lint rule from the `eslint-plugin-react-hooks` plugin, you can [analyze the effects as you type in your editor](https://github.com/facebook/react/issues/14920) and receive suggestions about which dependencies are missing. In other words, a machine can tell you which data flow changes aren’t handled correctly by a component.

![Lint rule gif](./exhaustive-deps.gif)

Pretty sweet.

## But I Can’t Put This Function Inside an Effect

Sometimes you might not want to move a function *inside* an effect. For example, several effects in the same component may call the same function, and you don’t want to copy and paste its logic. Or maybe it’s a prop.

Should you skip a function like this in the effect dependencies? I think not. Again, **effects shouldn’t lie about their dependencies.** There are usually better solutions. A common misconception is that “a function would never change”. But as we learned throughout this article, this couldn’t be further from truth. Indeed, a function defined inside a component changes on every render!

**That by itself presents a problem.** Say two effects call `getFetchUrl`:

```jsx
function SearchResults() {
  function getFetchUrl(query) {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }

  useEffect(() => {
    const url = getFetchUrl('react');
    // ... Fetch data and do something ...
  }, []); // 🔴 Missing dep: getFetchUrl

  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... Fetch data and do something ...
  }, []); // 🔴 Missing dep: getFetchUrl

  // ...
}
```

In that case you might not want to move `getFetchUrl` inside either of the effects since you wouldn’t be able to share the logic.

On the other hand, if you’re “honest” about the effect dependencies, you may run into a problem. Since both our effects depend on `getFetchUrl` **(which is different on every render)**, our dependency arrays are useless:

```jsx{2-5}
function SearchResults() {
  // 🔴 Re-triggers all effects on every render
  function getFetchUrl(query) {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }

  useEffect(() => {
    const url = getFetchUrl('react');
    // ... Fetch data and do something ...
  }, [getFetchUrl]); // 🚧 Deps are correct but they change too often

  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... Fetch data and do something ...
  }, [getFetchUrl]); // 🚧 Deps are correct but they change too often

  // ...
}
```

A tempting solution to this is to just skip the `getFetchUrl` function in the deps list. However, I don’t think it’s a good solution. This makes it difficult to notice when we *are* adding a change to the data flow that *needs* to be handled by an effect. This leads to bugs like the “never updating interval” we saw earlier.

Instead, there are two other solutions that are simpler.

**First of all, if a function doesn’t use anything from the component scope, you can hoist it outside the component and then freely use it inside your effects:**

```jsx{1-4}
// ✅ Not affected by the data flow
function getFetchUrl(query) {
  return 'https://hn.algolia.com/api/v1/search?query=' + query;
}

function SearchResults() {
  useEffect(() => {
    const url = getFetchUrl('react');
    // ... Fetch data and do something ...
  }, []); // ✅ Deps are OK

  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... Fetch data and do something ...
  }, []); // ✅ Deps are OK

  // ...
}
```

There’s no need to specify it in deps because it’s not in the render scope and can’t be affected by the data flow. It can’t accidentally depend on props or state.

Alternatively, you can wrap it into the [`useCallback` Hook](https://reactjs.org/docs/hooks-reference.html#usecallback):


```jsx{2-5}
function SearchResults() {
  // ✅ Preserves identity when its own deps are the same
  const getFetchUrl = useCallback((query) => {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }, []);  // ✅ Callback deps are OK

  useEffect(() => {
    const url = getFetchUrl('react');
    // ... Fetch data and do something ...
  }, [getFetchUrl]); // ✅ Effect deps are OK

  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... Fetch data and do something ...
  }, [getFetchUrl]); // ✅ Effect deps are OK

  // ...
}
```

`useCallback` is essentially like adding another layer of dependency checks. It’s solving the problem on the other end — **rather than avoid a function dependency, we make the function itself only change when necessary**.

Let's see why this approach is useful. Previously, our example showed two search results (for `'react'` and `'redux'` search queries). But let's say we want to add an input so that you can search for an arbitrary `query`. So instead of taking `query` as an argument, `getFetchUrl` will now read it from local state.

We'll immediately see that it's missing a `query` dependency:

```jsx{5}
function SearchResults() {
  const [query, setQuery] = useState('react');
  const getFetchUrl = useCallback(() => { // No query argument
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }, []); // 🔴 Missing dep: query
  // ...
}
```

If I fix my `useCallback` deps to include `query`, any effect with `getFetchUrl` in deps will re-run whenever the `query` changes:

```jsx{4-7}
function SearchResults() {
  const [query, setQuery] = useState('react');

  // ✅ Preserves identity until query changes
  const getFetchUrl = useCallback(() => {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }, [query]);  // ✅ Callback deps are OK

  useEffect(() => {
    const url = getFetchUrl();
    // ... Fetch data and do something ...
  }, [getFetchUrl]); // ✅ Effect deps are OK

  // ...
}
```

Thanks to `useCallback`, if `query` is the same, `getFetchUrl` also stays the same, and our effect doesn't re-run. But if `query` changes, `getFetchUrl` will also change, and we will re-fetch the data. It's a lot like when you change some cell in an Excel spreadsheet, and the other cells using it recalculate automatically.

This is just a consequence of embracing the data flow and the synchronization mindset. **The same solution works for function props passed from parents:**

```jsx{4-8}
function Parent() {
  const [query, setQuery] = useState('react');

  // ✅ Preserves identity until query changes
  const fetchData = useCallback(() => {
    const url = 'https://hn.algolia.com/api/v1/search?query=' + query;
    // ... Fetch data and return it ...
  }, [query]);  // ✅ Callback deps are OK

  return <Child fetchData={fetchData} />
}

function Child({ fetchData }) {
  let [data, setData] = useState(null);

  useEffect(() => {
    fetchData().then(setData);
  }, [fetchData]); // ✅ Effect deps are OK

  // ...
}
```

Since `fetchData` only changes inside `Parent` when its `query` state changes, our `Child` won’t refetch the data until it’s actually necessary for the app.

## Are Functions Part of the Data Flow?

Interestingly, this pattern is broken with classes in a way that really shows the difference between the effect and lifecycle paradigms. Consider this translation:

```jsx{5-8,18-20}
class Parent extends Component {
  state = {
    query: 'react'
  };
  fetchData = () => {
    const url = 'https://hn.algolia.com/api/v1/search?query=' + this.state.query;
    // ... Fetch data and do something ...
  };
  render() {
    return <Child fetchData={this.fetchData} />;
  }
}

class Child extends Component {
  state = {
    data: null
  };
  componentDidMount() {
    this.props.fetchData();
  }
  render() {
    // ...
  }
}
```

You might be thinking: “Come on Dan, we all know that `useEffect` is like `componentDidMount` and `componentDidUpdate` combined, you can’t keep beating that drum!” **Yet this doesn’t work even with `componentDidUpdate`:**

```jsx{8-13}
class Child extends Component {
  state = {
    data: null
  };
  componentDidMount() {
    this.props.fetchData();
  }
  componentDidUpdate(prevProps) {
    // 🔴 This condition will never be true
    if (this.props.fetchData !== prevProps.fetchData) {
      this.props.fetchData();
    }
  }
  render() {
    // ...
  }
}
```

Of course, `fetchData` is a class method! (Or, rather, a class property — but that doesn’t change anything.) It’s not going to be different because of a state change. So `this.props.fetchData` will stay equal to `prevProps.fetchData` and we’ll never refetch. Let’s just remove this condition then?

```jsx
  componentDidUpdate(prevProps) {
    this.props.fetchData();
  }
```

Oh wait, this fetches on *every* re-render. (Adding an animation above in the tree is a fun way to discover it.) Maybe let’s bind it to a particular query?

```jsx
  render() {
    return <Child fetchData={this.fetchData.bind(this, this.state.query)} />;
  }
```

But then `this.props.fetchData !== prevProps.fetchData` is *always* `true`, even if the `query` didn’t change! So we’ll *always* refetch.

The only real solution to this conundrum with classes is to bite the bullet and pass the `query` itself into the `Child` component. The `Child` doesn’t actually end up *using* the `query`, but it can trigger a refetch when it changes:

```jsx{10,22-24}
class Parent extends Component {
  state = {
    query: 'react'
  };
  fetchData = () => {
    const url = 'https://hn.algolia.com/api/v1/search?query=' + this.state.query;
    // ... Fetch data and do something ...
  };
  render() {
    return <Child fetchData={this.fetchData} query={this.state.query} />;
  }
}

class Child extends Component {
  state = {
    data: null
  };
  componentDidMount() {
    this.props.fetchData();
  }
  componentDidUpdate(prevProps) {
    if (this.props.query !== prevProps.query) {
      this.props.fetchData();
    }
  }
  render() {
    // ...
  }
}
```

Over the years of working with classes with React, I’ve gotten so used to passing unnecessary props down and breaking encapsulation of parent components that I only realized a week ago why we had to do it.

**With classes, function props by themselves aren’t truly a part of the data flow.** Methods close over the mutable `this` variable so we can’t rely on their identity to mean anything. Therefore, even when we only want a function, we have to pass a bunch of other data around in order to be able to “diff” it. We can’t know whether `this.props.fetchData` passed from the parent depends on some state or not, and whether that state has just changed.

**With `useCallback`, functions can fully participate in the data flow.** We can say that if the function inputs changed, the function itself has changed, but if not, it stayed the same. Thanks to the granularity provided by `useCallback`, changes to props like `props.fetchData` can propagate down automatically.

Similarly, [`useMemo`](https://reactjs.org/docs/hooks-reference.html#usememo) lets us do the same for complex objects:

```jsx
function ColorPicker() {
  // Doesn't break Child's shallow equality prop check
  // unless the color actually changes.
  const [color, setColor] = useState('pink');
  const style = useMemo(() => ({ color }), [color]);
  return <Child style={style} />;
}
```

**I want to emphasize that putting `useCallback` everywhere is pretty clunky.** It’s a nice escape hatch and it’s useful when a function is both passed down *and* called from inside an effect in some children. Or if you’re trying to prevent breaking memoization of a child component. But Hooks lend themselves better to [avoiding passing callbacks down](https://reactjs.org/docs/hooks-faq.html#how-to-avoid-passing-callbacks-down) altogether.

In the above examples, I’d much prefer if `fetchData` was either inside my effect (which itself could be extracted to a custom Hook) or a top-level import. I want to keep the effects simple, and callbacks in them don’t help that. (“What if some `props.onComplete` callback changes while the request was in flight?”) You can [simulate the class behavior](#swimming-against-the-tide) but that doesn’t solve race conditions.

## Speaking of Race Conditions

A classic data fetching example with classes might look like this:

```jsx
class Article extends Component {
  state = {
    article: null
  };
  componentDidMount() {
    this.fetchData(this.props.id);
  }
  async fetchData(id) {
    const article = await API.fetchArticle(id);
    this.setState({ article });
  }
  // ...
}
```

As you probably know, this code is buggy. It doesn’t handle updates. So the second classic example you could find online is something like this:

```jsx{8-12}
class Article extends Component {
  state = {
    article: null
  };
  componentDidMount() {
    this.fetchData(this.props.id);
  }
  componentDidUpdate(prevProps) {
    if (prevProps.id !== this.props.id) {
      this.fetchData(this.props.id);
    }
  }
  async fetchData(id) {
    const article = await API.fetchArticle(id);
    this.setState({ article });
  }
  // ...
}
```

This is definitely better! But it’s still buggy. The reason it’s buggy is that the request may come out of order. So if I’m fetching `{id: 10}`, switch to `{id: 20}`, but the `{id: 20}` request comes first, the request that started earlier but finished later would incorrectly overwrite my state.

This is called a race condition, and it’s typical in code that mixes `async` / `await` (which assumes something waits for the result) with top-down data flow (props or state can change while we’re in the middle of an async function).

Effects don’t magically solve this problem, although they’ll warn you if you try to pass an `async` function to the effect directly. (We’ll need to improve that warning to better explain the problems you might run into.)

If the async approach you use supports cancellation, that’s great! You can cancel the async request right in your cleanup function.

Alternatively, the easiest stopgap approach is to track it with a boolean:

```jsx{5,9,16-18}
function Article({ id }) {
  const [article, setArticle] = useState(null);

  useEffect(() => {
    let didCancel = false;

    async function fetchData() {
      const article = await API.fetchArticle(id);
      if (!didCancel) {
        setArticle(article);
      }
    }

    fetchData();

    return () => {
      didCancel = true;
    };
  }, [id]);

  // ...
}
```

[This article](https://www.robinwieruch.de/react-hooks-fetch-data/) goes into more detail about how you can handle errors and loading states, as well as extract that logic into a custom Hook. I recommend you to check it out if you’re interested to learn more about data fetching with Hooks.

## Raising the Bar

With the class lifecycle mindset, side effects behave differently from the render output. Rendering the UI is driven by props and state, and is guaranteed to be consistent with them, but side effects are not. This is a common source of bugs.

With the mindset of `useEffect`, things are synchronized by default. Side effects become a part of the React data flow. For every `useEffect` call, once you get it right, your component handles edge cases much better.

However, the upfront cost of getting it right is higher. This can be annoying. Writing synchronization code that handles edge cases well is inherently more difficult than firing one-off side effects that aren’t consistent with rendering.

This could be worrying if `useEffect` was meant to be *the* tool you use most of the time. However, it’s a low-level building block. It’s an early time for Hooks so everybody uses low-level ones all the time, especially in tutorials. But in practice, it’s likely the community will start moving to higher-level Hooks as good APIs gain momentum.

I’m seeing different apps create their own Hooks like `useFetch` that encapsulates some of their app’s auth logic or `useTheme` which uses theme context. Once you have a toolbox of those, you don’t reach for `useEffect` *that* often. But the resilience it brings benefits every Hook built on top of it.

So far, `useEffect` is most commonly used for data fetching. But data fetching isn’t exactly a synchronization problem. This is especially obvious because our deps are often `[]`. What are we even synchronizing?

In the longer term, [Suspense for Data Fetching](https://reactjs.org/blog/2018/11/27/react-16-roadmap.html#react-16x-mid-2019-the-one-with-suspense-for-data-fetching) will allow third-party libraries to have a first-class way to tell React to suspend rendering until something async (anything: code, data, images) is ready.

As Suspense gradually covers more data fetching use cases, I anticipate that `useEffect` will fade into background as a power user tool for cases when you actually want to synchronize props and state to some side effect. Unlike data fetching, it handles this case naturally because it was designed for it. But until then, custom Hooks like [shown here](https://www.robinwieruch.de/react-hooks-fetch-data/) are a good way to reuse data fetching logic.

## In Closing

Now that you know pretty much everything I know about using effects, check out the [TLDR](#tldr) in the beginning. Does it make sense? Did I miss something? (I haven’t run out of paper yet!)

I’d love to hear from you on Twitter! Thanks for reading.
