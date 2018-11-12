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

## Рендер списка фильмов

Теперь попробуем отобразить список фильмов. Для этого нам нужно:

1. Новое поле в стейте `state = { value: '', movies: [] }`. `movies` - это массив фильмов

2. Отображение наших данных из стейта внутри рендера:

```
{this.state.movies.map(movie => (
  <div key={movie.id}>{movie.title}</div>
))}
```

Здесь мы проходим по массиву данных map'ом и возвращаем новый массив дивов, который реакт нам благополучно отрендерит. Сам `movie` — это здоровый объект, в котором есть куча ненужных полей. Из них мы заберем id, уникальный для каждого фильма, для атррибута key. Он нужен реакту, чтобы тот не путался, и понимал, какой из компонентов нужно ререндерить, а какой — нет. Также мы забрем `movie.title` — это просто строка, имя фильма.

<img src="https://raw.githubusercontent.com/Fivapr/Movie-Searcher-Book/master/response.png" width=500 alt="response json" />

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

## Обновление списка фильмов после каждой смены инпута

Чтобы обновлять список фильмов синхронно с ченжем инпута, нам либо после каждого нажатия клавиши (onChange евента) или после каждого setState нужно вызывать запрос, содержащий строку поиска, ждать его ресолва и после этого так же как на дидМаунте класть в стейт фильмы, а дальше они сами отрендерятся.

Для этого мы должны немного поменять строку запроса из дидМаунта, `movie/top_rated` на `search/movie`, и вставить в нее параметр query=\${this.state.value}. Тогда, на каждое изменение `this.state.value` мы отправляли бы новый запрос поиска фильма и получали новый набор фильмов.

Если бы мы делали этот запрос сразу после `setState`, то код ниже бы не сработал правильно

```
  onChange = e => {
    this.setState({ value: e.target.value })
    axios
      .get(`https://api.themoviedb.org/3/search/movie?api_key=59017ce86d5101576f32f47160168519&query=${this.state.value}`)
      .then(res => this.setState({ movies: res.data.results }))
      .catch(err => console.log(err))
  }
```

тк функция setState асинхронная, и соответственно в строке запроса `this.state.value` был бы равен старому значению `value`, тому, которое было бы до запуска и `setState`, и `onChage` функции. Это происходит потому что setState еще не успел присвоить значение`e.target.value` нашему стейту. В этом случае, все запросы поиска будут отставать от того, что мы реально хотим найти на 1 символ (можете сами проверить такое поведение с кодом выше). Чтобы не было такого рода багов мы положим этот код во [второй аргумент функции setState](https://reactjs.org/docs/react-component.html#setstate). Это функция, которую мы кладем вторым аргументом, гарантированно срабатывает после того, как setState законсил присваивание нового стейта.

Таким образом новая функция `onChange` будет выглядеть так:

```
  onChange = e => {
    this.setState({ value: e.target.value }, () =>
      axios
        .get(`https://api.themoviedb.org/3/search/movie?api_key=59017ce86d5101576f32f47160168519&query=${this.state.value}`)
        .then(res => this.setState({ movies: res.data.results }))
        .catch(err => console.log(err))
    )
  }
```

Теперь мы на каждое изменение в инпуте получаем новые фильмы!

## Material-UI

Хм, наш поиск фильмов сейчас максимально всратый, чтобы сделать его красивее, добавим в проект юи-фреймворк, из которого будем брать готовые красивые компоненты

```
yarn add @material-ui/core
```

далее копипастим код длинного инпута на весь экран [отсюда](https://material-ui.com/demos/text-fields/#textfield), не забывая заипортить TextField сверху файла.

```
        <TextField
          id="standard-full-width"
          label="Label"
          style={{ margin: 8 }}
          placeholder="Placeholder"
          helperText="Full width!"
          fullWidth
          margin="normal"
          InputLabelProps={{
            shrink: true,
          }}
        />
```

И заменяем этим `TextField` наш старый `input`, но при этом добавляя в новый, красивый компонент логику старого, те значения `value` и функцию `onChange`. И удаляем какие-то ненужные пропсы, чтобы было почище.

```
        <TextField
          label="Search for the movie!"
          placeholder="La la land"
          fullWidth
          margin="normal"
          InputLabelProps={{
            shrink: true
          }}
          value={this.state.value}
          onChange={this.onChange}
        />
```

Теперь нам нужно отобразить кроме самих названий фильмов большие карточки с постером фильма, названием и кратким описанием. Для этого скопипастим из MUI [вот такой вот компонент](https://material-ui.com/demos/cards/#media) и вставим его внутрь нашей фкнции map фильмов

```
          {this.state.movies.map(movie => (
            <Card className={classes.card}>
              <CardActionArea>
                <CardMedia
                  className={classes.media}
                  image="/static/images/cards/contemplative-reptile.jpg"
                  title="Contemplative Reptile"
                />
                <CardContent>
                  <Typography gutterBottom variant="h5" component="h2">
                    Lizard
                  </Typography>
                  <Typography component="p">
                    Lizards are a widespread group of squamate reptiles, with over 6,000 species, ranging
                    across all continents except Antarctica
                  </Typography>
                </CardContent>
              </CardActionArea>
              <CardActions>
                <Button size="small" color="primary">
                  Share
                </Button>
                <Button size="small" color="primary">
                  Learn More
                </Button>
              </CardActions>
            </Card>
          ))}
```

Чтобы этот пример не сломался нам нужно скопипастить еще стили и зависимости MUI, и в конце обернуть наш экспорт в HOC `withStyles(styles)`, он добавляет этот объект `const styles`, в качестве пропсов в наш компонент. На деле можно было бы просто прописать стили руками в компоненте, но с названиями это выразительней... В общем так решили челики из MUI, и так вроде как удобно инжектить темы в стили. В общем HOC'и мне пока лень объяснять.

```
import { withStyles } from '@material-ui/core/styles';
import Card from '@material-ui/core/Card';
import CardActionArea from '@material-ui/core/CardActionArea';
import CardActions from '@material-ui/core/CardActions';
import CardContent from '@material-ui/core/CardContent';
import CardMedia from '@material-ui/core/CardMedia';
import Button from '@material-ui/core/Button';
import Typography from '@material-ui/core/Typography';

...

const styles = {
  card: {
    maxWidth: 345,
  },
  media: {
    height: 140,
  },
};

export default withStyles(styles)(App);
```

Сейчас мы видим 20 карточек (по количеству фильмов, которые нам приходят на одну страницу), на которых есть две кнопки, которые нам не нужны, мы их удаляем, какой-то текст про жаб, который мы заменим на `{movie.overview}`, заголовок, который мы поменяем на `{movie.title}` и ссылку на пикчу мы поменяем на полный урл `` image={`http://image.tmdb.org/t/p/w500/${movie.poster_path}`} ``. В объекте фильма нам приходит только последняя часть ссылки, поэтому мы дописываем начало самии вставляем концовку как переменную на каждый фильм.

В общем в итоге получим, вот такой вот код карточек. И чтобы они не были располежены друг под другом, зададим дивом простейшую flex-сетку и немного поменяем стили, чтобы фотографии карточек помещались в них не полностью.

```
const styles = {
  card: {
    maxWidth: 345,
    margin: 10
  },
  media: {
    height: 500
  }
}

...

        <div style={{ display: 'flex', flexWrap: 'wrap', justifyContent: 'space-around' }}>
          {this.state.movies.map(movie => (
            <Card className={classes.card} key={movie.id}>
              <CardActionArea>
                <CardMedia
                  className={classes.media}
                  image={`http://image.tmdb.org/t/p/w500/${movie.poster_path}`}
                  title="Contemplative Reptile"
                />
                <CardContent>
                  <Typography gutterBottom variant="h5" component="h2">
                    {movie.title}
                  </Typography>
                  <Typography component="p">{movie.overview}</Typography>
                </CardContent>
              </CardActionArea>
            </Card>
          ))}
        </div>
```

В конце получим вот такой вот компонент, и это и будет первая модель мувисерчера всего на 100 строк!

```
import React, { Component } from 'react'
import axios from 'axios'
import TextField from '@material-ui/core/TextField'
import { withStyles } from '@material-ui/core/styles'
import Card from '@material-ui/core/Card'
import CardActionArea from '@material-ui/core/CardActionArea'
import CardContent from '@material-ui/core/CardContent'
import CardMedia from '@material-ui/core/CardMedia'
import Typography from '@material-ui/core/Typography'

const styles = {
  card: {
    maxWidth: 345,
    margin: 10
  },
  media: {
    height: 500
  }
}

class App extends Component {
  state = { value: '', movies: [] }
  onChange = e => {
    this.setState({ value: e.target.value }, () =>
      axios
        .get(
          `https://api.themoviedb.org/3/search/movie?api_key=59017ce86d5101576f32f47160168519&query=${
            this.state.value
          }`
        )
        .then(res => this.setState({ movies: res.data.results }))
        .catch(err => console.log(err))
    )
  }

  componentDidMount() {
    axios
      .get('https://api.themoviedb.org/3/movie/top_rated?api_key=59017ce86d5101576f32f47160168519')
      .then(res => this.setState({ movies: res.data.results }))
      .catch(err => console.log(err))
  }

  render() {
    const { classes } = this.props
    return (
      <>
        <TextField
          label="Search for the movie!"
          placeholder="La la land"
          fullWidth
          margin="normal"
          InputLabelProps={{
            shrink: true
          }}
          value={this.state.value}
          onChange={this.onChange}
        />

        <div style={{ display: 'flex', flexWrap: 'wrap', justifyContent: 'space-around' }}>
          {this.state.movies.map(movie => (
            <Card className={classes.card} key={movie.id}>
              <CardActionArea>
                <CardMedia
                  className={classes.media}
                  image={`http://image.tmdb.org/t/p/w500/${movie.poster_path}`}
                  title="Contemplative Reptile"
                />
                <CardContent>
                  <Typography gutterBottom variant="h5" component="h2">
                    {movie.title}
                  </Typography>
                  <Typography component="p">{movie.overview}</Typography>
                </CardContent>
              </CardActionArea>
            </Card>
          ))}
        </div>
      </>
    )
  }
}

export default withStyles(styles)(App)

```

## Рефакторинг

## Деплой с помощью now

Чтобы задеплоить свой невероятный мувисерчер мы глобально установим now.

```
npm i -g now
```

Он позволяет задеплоить апп вообще без настроек одной командой из рута вашего проекта. Правда вам еще надо будет зарегистрировать и подтвердить ящик.

```
now
```

Это займет около минуты, и now вам выдаст какое-то всратое название места, куда вы задеплоили уровня moviesearcher-scnsedikmv.now.sh. Чтобы его сменить, нужно выполнить команду `now alias`

```
now alias moviesearcher-scnsedikmv.now.sh moviesearcher
```

По ее завершению ваш серчер будет доступен по адресу moviesearcher.now.sh. А чтобы не вводить эту команду при каждом деплое мы добавим файл `now.json` с настройками деплоя. В него мы добавим одну строку.

```
{
  "alias": "moviesearcher"
}
```

Теперь когда мы запускаем now alias, мы можем не вводить параметры. Он просто возьмет последний задеплоенный урл первым параметром и `alias` из `now.json` вторым. Но и это мне вводить несколько лень.
Поэтому в разделе скриптов package.json добавим скрипт:

```
  "now": "now && now alias"
```

Теперь мы можем запускать весь этот процесс командой.

```
yarn now
```