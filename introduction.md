---
title: Introduction
seoTitle: moviesearcher app
seoDescription: moviesearcher app for frontend thread
isFree: true
---

## Введение или что такое мувисерчер и для чего эта книга?

Мувисерчер — тестовое SPA после которого каждый анон может идти покорять эйчарок. С его помощью вы сможете найти работу до 50к/наносек в своем миллионнике или до 80к/наносек в ДСе. Сегодня и только сегодня первая вводная глава бесплатна — это тизер обширного туториала по миру фронтэнда, цена которого будет всего 10$. 10$ сегодня, 1000\$ заработка уже через месяц!

Наш мувисерчер будет построен с помощью либ React, Redux, React-Router Redux-Saga, ImmutableJS, Reselect, Redux-symbiote, Material-UI,Styled-components и нескольких других полезных либ. Функционал покроет обычный поиск, страничку фильма, поиск по фильтрам и добавление любимых фильмов в локалстораж. Сначала мы напишем простейший серчер по строке на реакте, потом перенесем стейт в редакс и поочередно будем включать и рефакторить наш код с множеством новых либ.

## React

Прежде, чем начинать пилить серчер, тебя может заинтересовать [дока реакта](https://reactjs.org/docs/getting-started.html) и, откуда бы вы их не взяли, знания жаваскрипта и ес6 фич типа классов, промизов и стрелочных функций.

А теперь начнем.

Чтобы забутстрапить проект и не ковыряться в вебпаке мы использовуем create-react-app. Просто устанавливаем ноду и вводим в терминал/cmd:

```
npx create-react-app moviesearcher
```

И получаем вот такую файловую структуру.
<img src="https://raw.githubusercontent.com/Fivapr/Movie-Searcher-Book/master/cra-file-structure.png" width=250 alt="Пикча структуры" />

Можете насладиться тем, что хотрелоад работает из коробки набрав в консоли:

```
yarn start
```

И поменяв содержимое App.js на простейший компонент, который отрендерит один див:

```
import React, { Component } from 'react';

class App extends Component {
  render() {
    return (
      <div>moviesearcher</div>
    );
  }
}

export default App;
```

Продолжим очистку дефолтного CRA и удалим все остальные файлы кроме src/index.js, src/App.js и public/index.html.
<img src="https://raw.githubusercontent.com/Fivapr/Movie-Searcher-Book/master/clean-file-structure.png" width=250 alt="Пикча структуры" />

В public/index.html оставим только самое нужное, а именно `<div id="root"></div>`, `id="root"` — тот самый айдишник, в который реакт будет рендерить весь апп.

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />
    <title>Moviesearcher</title>
  </head>
  <body>
    <noscript> You need to enable JavaScript to run this app. </noscript>
    <div id="root"></div>
  </body>
</html>
```

В src/index.js оставим следующее:

```
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

ReactDOM.render(<App />, document.getElementById('root'));
```

Здесь мы импортим `ReactDOM.render`, чтобы в див, с айди 'root' заинжектить весь наш `<App />`, который мы заимпортили из соседнего файла. А импорт реакта нам здесь нужен для поддержки синтаксиса jsx, ака `<App />`. В итоге получаем чистый проект с тремя файлами. В следующей главе мы будем работать только с `App.js`.

## Поиск фильмов и работа с moviedb api

#### Controlled input

Первым делом, чтобы искать фильмы нужен инпут, добавим [controlled input](https://reactjs.org/docs/forms.html#controlled-components).
Для этого нам нужно:

1. Создать внутри нашего класса объект стейт с полем value, в котором мы будем хранить данные инпута. После объявления он будет доступен внутри класса через `this.state.value`. Где this — это инстанс нашего класса, а `state` - обычный объект. Заметьте, что мы можем с новым синтаксисом полей классов не использовать constructor, а так и писать `state =`. Они в итоге скомпилятся в один и тот же es5 код.

2. Создать обычный `input` c атрибутом `value={this.state.value}`. Теперь как бы вы не пытались поменять значение в браузере, оно не изменится без ведома реакта, и значение инпута всегда будет соответствовать его значению из стейта компонента. Вы можете поменять `state = { value: 'bleh-bleh-bleh' }`, и оно изменится в браузере (после хот-релоада). И не будет меняться обычными эвентами браузера.

3. Сделать так, чтобы реакт реагировал на евенты инпута из браузера. Для этого добавим инпуту атрибут `onChange={this.onChange}`. И также как со стейтом, напишем просто `onChange =`; используя стрелочную функцию, мы внутри тела функции можем использовать this равный инстансу нашего класса. Грубо говоря, чтобы понять, чем является this стрелочной функции, мы просто смотрим вокруг нее по коду, и то что мы видим - и есть this. В нашем случае функция располагается прямо в классе, и инстанс этого класса и будет this'ом для нашей функции.

4. В этой функции мы возьмем e, событие браузера, в нашем случае - change, возьмем из него target, который является зафокушенным во время события элементом. И из него заберем value. Присвоим это значение из евента нашему полю value из стейта с помощью специальной функции `setState`, которая как и много других методов идет с классом Component. Хочу заметить, что `state` - это не какое-то специальное зарезервированное имя, вы можете называть переменные класса как угодно, но функция setState обновляет именно `state` и триггерит ререндер, поэтому меняющиеся данные стоит хранить именно в нем.

5. Можете сделать `console.log(e)` или `console.log(this)` вместо setState'а, чтобы разобраться, что внутри евента, и чем является this у onChange функции (или лучше написать `debugger;` и поучиться работать с хромдевтулсами).

В итоге получаем вот такой `App.js`

```
import React, { Component } from 'react'

class App extends Component {
  state = { value: '' }
  onChange = e => this.setState({ value: e.target.value })

  render() {
    return (
      <>
        <input value={this.state.value} onChange={this.onChange} />
      </>
    )
  }
}

export default App

```

#### Рендер списка фильмов

Теперь попробуем отобразить список фильмов. Для этого нам нужно:

1. Новое поле в стейте `state = { value: '', movies: [] }`. `movies` - это массив фильмов

2. Отображение наших данных из стейта внутри рендера:

```
{this.state.movies.map(movie => (
  <div key={movie.id}>{movie.title}</div>
))}
```

Здесь мы проходим по массиву данных map'ом и возвращаем новый массив дивов, который реакт нам благополучно отрендерит. Сам `movie` — это здоровый объект, в котором есть куча ненужных полей. Из них мы заберем id, уникальный для каждого фильма, для атррибута key. Он нужен реакту, чтобы тот не путался, и понимал, какой из компонентов нужно ререндерить, а какой — нет. Также мы забрем `movie.title` — это просто строка, имя фильма.

<img src="https://raw.githubusercontent.com/Fivapr/Movie-Searcher-Book/master/response.png" width=620 alt="response json" />

3. Теперь нам нужны сами фильмы, сейчас мы рендерим пустой массив. Чтобы получить фильмы, нам нужно отправить запрос к tmdb api, забрать из ответа массив фильмов и положить в `state.movies`, после чего реакт реактивно (самостоятельно) их отрендерит.

Для удобного отправления запросов установим небольшую либу

```
yarn add axios
```

И внутри компонента добавим такой код

```
  componentDidMount() {
    axios
      .get('https://api.themoviedb.org/3/movie/top_rated?api_key=59017ce86d5101576f32f47160168519')
      .then(res => this.setState({ movies: res.data.results }))
      .catch(err => console.log(err))
  }
```

componentDidMount — это lifecycle hook, он позволяет на определенной стадии жизни компонента исполнять любые действия, в нашем случае отправление запроса. Функция DidMount исполняется сразу после того как компонент первый раз рендерится в DOM. То есть сразу после рендера мы делаем запрос, ждем ответа, кладем нужный нам кусок ответа в стейт и реакт его рендерит.

`.get('https://api.themoviedb.org/3/movie/top_rated?api_key=59017ce86d5101576f32f47160168519')` В этой части мы делаем асинхронный запрос к апи:

- `https://api.themoviedb.org/3/` — каждый запрос к тмдб апи будет отправляться именно сюда
- `movie/` — эта часть значит, что мы хотим от них именно фильмы
- `top_rated` — это главная часть запроса, нам пришлют 20 первых фильмов с самым высоким рейтингом.
- `?api_key=59017ce86d5101576f32f47160168519` — после вопроса мы пишем так называемые params'ы, далее они парсятся на сервере тмдб, чтобы он мог понять, что конкретно мы от него хотим. api_key нужно получить на сайте [themoviedb](https://themoviedb.org/), его мы должны прикреплять к каждому запросу, потому что тмдб не хочет, чтобы кто попало отправлял запросы и тратил их деньги.

- `.then(res => this.setState({ movies: res.data.results }))` — как только нам приходит ответ мы просто кладем его в стейт и он автоматически рендерится. Ошибки мы пока особым образом не обрабатываем, просто показываем их в консоли.

Не забудем добавить импорт axios'а и получим такой код компонента. Vois la. Теперь у нас при первом рендере появляется список фильмов!

```
import React, { Component } from 'react'
import axios from 'axios'

class App extends Component {
  state = { value: '', movies: [] }
  onChange = e => this.setState({ value: e.target.value })

  componentDidMount() {
    axios
      .get('https://api.themoviedb.org/3/movie/top_rated?api_key=59017ce86d5101576f32f47160168519')
      .then(res => this.setState({ movies: res.data.results }))
      .catch(err => console.log(err))
  }

  render() {
    return (
      <>
        <input value={this.state.value} onChange={this.onChange} />
        {this.state.movies.map(movie => (
          <div key={movie.id}>{movie.title}</div>
        ))}
      </>
    )
  }
}

export default App

```
