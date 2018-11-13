---
title: Introduction
seoTitle: moviesearcher app
seoDescription: moviesearcher app for frontend thread
isFree: true
---

## Введение или что такое мувисерчер и для чего эта книга?

Мувисерчер — тестовое SPA, после которого каждый анон может идти покорять эйчарок. С его помощью вы сможете найти работу до 50к/наносек в своем миллионнике или до 80к/наносек в ДСе. Сегодня и только сегодня первая вводная глава бесплатна — это тизер обширного туториала по миру фронтэнда, цена которого будет всего 10$. 10$ сегодня, 1000\$ заработка уже через месяц!

Наш мувисерчер будет построен с помощью либ React, Redux, React-Router Redux-Saga, ImmutableJS, Reselect, Redux-symbiote, Material-UI,Styled-components и нескольких других полезных либ. Функционал покроет обычный поиск, страничку фильма, поиск по фильтрам и добавление любимых фильмов в локалстораж. Сначала мы напишем простейший серчер по строке на реакте, потом перенесем стейт в редакс и поочередно будем включать новые возможности и рефакторить наш код, применяя best practices.

Если вас заинтересует, то текущий код серчера можно найти в [этой репе](https://github.com/Fivapr/Movie-Searcher-App)
А текущий инстанс задеплоен по адресу [moviesearcher.app](moviesearcher.app)

## React

Прежде, чем начинать пилить серчер, тебя может заинтересовать [дока реакта](https://reactjs.org/docs/getting-started.html) и, откуда бы ты их не взял, знания жаваскрипта и ес6 фич типа классов, промизов и стрелочных функций.

А теперь начнем.

Чтобы забутстрапить проект и не ковыряться в вебпаке мы использовуем create-react-app. Просто устанавливаем ноду и вводим в терминал/cmd:

```
npx create-react-app moviesearcher
```

И получаем вот такую файловую структуру.
<img src="https://raw.githubusercontent.com/Fivapr/Movie-Searcher-Book/master/cra-file-structure.png" width=245 alt="Пикча структуры" />

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
<img src="https://raw.githubusercontent.com/Fivapr/Movie-Searcher-Book/master/clean-file-structure.png" width=246 alt="Пикча структуры" />

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

Здесь мы импортим `ReactDOM.render`, чтобы в див, с `id="root"` заинжектить весь наш `<App />`, который мы заимпортили из соседнего файла. А импорт реакта нам здесь нужен для поддержки синтаксиса jsx, ака `<App />`. В итоге получаем чистый проект с тремя файлами. В следующей главе мы будем работать только с `App.js`.

## Поиск фильмов и работа с moviedb api

Первым делом, чтобы искать фильмы нужен инпут, добавим [controlled input](https://reactjs.org/docs/forms.html#controlled-components).
Для этого нам нужно:

1. Создать внутри нашего класса объект стейт с полем value, в котором мы будем хранить данные инпута. После объявления он будет доступен внутри класса через `this.state.value`. Где this — это инстанс нашего класса, а `state` - обычный объект. Заметьте, что мы можем с новым синтаксисом полей классов не использовать `constructor`, а так и писать `state =` . Они в итоге скомпилятся в один и тот же es5 код.

2. Создать обычный `input` c атрибутом `value={this.state.value}`. Теперь как бы вы не пытались поменять значение в браузере, оно не изменится без ведома реакта, и значение инпута всегда будет соответствовать его значению из стейта компонента. Вы можете поменять `state = { value: 'bleh-bleh-bleh' }`, и оно изменится в браузере (после хот-релоада). И не будет меняться обычными эвентами браузера.

3. Сделать так, чтобы реакт реагировал на евенты инпута из браузера. Для этого добавим атрибут `onChange={this.onChange}`. И также как со стейтом, напишем просто `onChange =`. Используя стрелочную функцию, мы внутри тела функции можем использовать this равный инстансу нашего класса. Грубо говоря, чтобы понять, чем является this стрелочной функции, мы просто смотрим вокруг нее по коду, и то что мы видим - и есть this. В нашем случае функция располагается прямо в классе, и инстанс этого класса и будет this'ом для нашей функции.

4. В этой функции мы возьмем `e`, событие браузера, в нашем случае - change, возьмем из него target, который является зафокушенным во время события элементом. И из него заберем value. Присвоим это значение из евента нашему полю value из стейта с помощью специальной функции `setState`, которая как и много других методов идет с классом Component. Хочу заметить, что `state` - это не какое-то специальное зарезервированное имя, вы можете называть переменные класса как угодно, но функция setState обновляет именно `state`, триггерит ререндер и запускает весь [reconciliation process](https://reactjs.org/docs/reconciliation.html), поэтому меняющиеся данные стоит хранить именно в нем.

5. Можете сделать `console.log(e)` или `console.log(this)` вместо setState'а, чтобы разобраться, что внутри евента, и чем является this у onChange функции (еще можно написать `debugger;` и поучиться работать с хромдевтулсами).

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

Кстати говоря `<> </>` — это сокращенный синтаксис для [React.Fragment](https://reactjs.org/docs/fragments.html). Он просто позволяет не писать лишних дивов-контейнеров там, где это не нужно и сохраняет DOM чистым.

## Рендер списка фильмов

Теперь попробуем отобразить список фильмов. Для этого нам нужно:

1. Новое поле в стейте `state = { value: '', movies: [] }`. `movies` - это массив фильмов. Мы ставим ему дефолтное значение `[]`, чтобы программа не ломалась без данных на первом рендере, так как далее мы используем метод из прототипа массива `map`. Советую всегда задавать дефолтные значения и следить, чтобы далее в код не вылез эксепшн несуществующего метода или проперти.

2. Отображение наших данных из стейта внутри рендера:

```
{this.state.movies.map(movie => (
  <div key={movie.id}>{movie.title}</div>
))}
```

Здесь мы проходим по массиву данных map'ом и возвращаем новый массив дивов, который реакт нам благополучно отрендерит. Сам `movie` — это здоровый объект, в котором есть куча нужных и ненужных полей. Из них сейчас мы заберем `id`, уникальный для каждого фильма, для атрибута key. Он нужен реакту, чтобы тот не путался, и понимал, какой из компонентов нужно ререндерить, а какой — нет. Также мы забрем `movie.title` — это просто строка, имя фильма.

3. Теперь нам нужны сами фильмы, сейчас мы рендерим пустой массив. Чтобы получить фильмы, нам нужно отправить запрос к tmdb api, забрать из ответа массив фильмов и положить в `state.movies` с помощью `setState`, после чего реакт реактивно (самостоятельно) их отрендерит. Пока что мы будем рендерить только тайтлы, но дальше нам потребуется еще ссылка на постер и краткое описание, оба они есть среди полей ответа. Так будет выглядеть массив фильмов в девтулсах.

<img src="https://raw.githubusercontent.com/Fivapr/Movie-Searcher-Book/master/response.png" width=606 height=530 alt="response json" />

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

componentDidMount — это lifecycle hook, он позволяет на определенной стадии жизни компонента исполнять любые действия, в нашем случае отправление запроса. Функция DidMount исполняется сразу после того как компонент первый раз рендерится в DOM. То есть сразу после маунта мы делаем запрос, ждем ответа, кладем нужный нам кусок ответа в стейт и реакт повторно ререндерит компонент с новым стейтом.

`.get('https://api.themoviedb.org/3/movie/top_rated?api_key=59017ce86d5101576f32f47160168519')` В этой части мы делаем асинхронный запрос к апи:

- `https://api.themoviedb.org/3/` — каждый запрос к тмдб апи будет отправляться именно сюда
- `movie/` — эта часть значит, что мы хотим от них именно фильмы
- `top_rated` — это главная часть запроса, нам пришлют 20 первых фильмов с самым высоким рейтингом.
- `?api_key=59017ce86d5101576f32f47160168519` — после вопроса мы пишем так называемые params'ы, далее они парсятся на сервере тмдб, чтобы он мог понять, что конкретно мы от него хотим. api_key нужно получить на сайте [themoviedb](https://themoviedb.org/), его мы должны прикреплять к каждому запросу, потому что тмдб не хочет, чтобы кто попало отправлял запросы и тратил их деньги.

- `.then(res => this.setState({ movies: res.data.results }))` — как только нам приходит ответ мы просто кладем его в стейт и он автоматически рендерится. Напомню, что `.then` работает асинхронно, те только после того, как аксиос послал запрос и ему пришел ответ промис ресолвится, и затем срабатывает `.then`. Ошибки мы пока особым образом не обрабатываем, просто показываем их в консоли.

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

Для этого мы должны немного поменять строку запроса из дидМаунта, `movie/top_rated` на `search/movie`, и вставить в нее параметр `query=${this.state.value}`. Тогда, на каждое изменение `this.state.value` мы отправляли бы новый запрос поиска фильма и получали новый набор фильмов.

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

тк функция setState асинхронная, и соответственно в строке запроса `this.state.value` был бы равен старому значению `value`, тому, которое было бы до запуска и `setState`, и `onChage` функции. Это происходит потому что setState еще не успел присвоить значение`e.target.value` нашему стейту. В этом случае, все запросы поиска будут отставать от того, что мы реально хотим найти на 1 символ (можете сами проверить такое поведение с кодом выше). Чтобы не было такого рода багов мы положим этот код во [второй аргумент функции setState](https://reactjs.org/docs/react-component.html#setstate). Это функция — коллбэк, она гарантированно срабатывает после того, как setState законсил присваивание нового стейта.

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

далее копипастим код длинного инпута на весь экран [отсюда](https://material-ui.com/demos/text-fields/#textfield), не забывая заимпортить TextField сверху файла.

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

Теперь нам нужно отобразить кроме самих названий фильмов большие карточки с постером фильма, названием и кратким описанием. Для этого скопипастим из MUI [вот такой вот компонент](https://material-ui.com/demos/cards/#media) и вставим его внутрь нашей функции `map` фильмов

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

Чтобы этот пример не сломался нам нужно скопипастить еще стили и зависимости MUI, и в конце обернуть наш экспорт в [HOC](https://reactjs.org/docs/higher-order-components.html) `withStyles(styles)`, он добавляет этот объект `const styles`, в качестве пропсов в наш компонент так, чтобы могли использовать их в качестве `className`. На деле можно было бы просто прописать стили руками в компоненте, но с названиями это выразительней... В общем так решили челики из MUI, и так вроде как удобно инжектить темы в стили.

Сам HOC — Higher order component это функция, которая возвращает компонент. В большинстве своем HOC'и просто вставляют новые пропсы в компонент и обычно называются withStyles, withConnect, withReducer, и тд.

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

Сейчас мы видим 20 карточек (по количеству фильмов, которые нам приходят на одну страницу), на которых есть две кнопки, которые нам не нужны, мы их удаляем, какой-то текст про жаб, который мы заменим на `{movie.overview}`, заголовок, который мы поменяем на `{movie.title}` и ссылку на пикчу мы поменяем на полный урл `` image={`http://image.tmdb.org/t/p/w500/${movie.poster_path}`} ``. В объекте фильма нам приходит только последняя часть ссылки, поэтому мы дописываем начало сами и вставляем концовку как переменную на каждый фильм.

В итоге получим, вот такой вот код карточек. И чтобы они не были располежены друг под другом, зададим дивом простейшую flex-сетку и немного поменяем стили, чтобы фотографии карточек помещались в них полностью.

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
          image={`https://image.tmdb.org/t/p/w500/${movie.poster_path}`}
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

В конце получим вот такой вот компонент, это и будет первая модель мувисерчера всего на 100 строк!

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
                  image={`https://image.tmdb.org/t/p/w500/${movie.poster_path}`}
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

Чтож, наш серчер разросся, а это значит, что пора бы выделить функции и сделать код [DRY](https://ru.wikipedia.org/wiki/Don%E2%80%99t_repeat_yourself).

- Во-первых у нас повторяются куски урла в запросе аксиоса, все эти длинные строки выглядят максимально всрато, мы можем оберунуть весь этот запрос в функцию, и вызывать что-то вроде `api('movie/top_rated')`, а она уже сконкатенирует строки как нам надо. Также создадим файл `config.js` для глобальных переменных вроде api_key и базового урл для запросов.

- Вообще вот так в открытую писать в коде api_key не очень безопасно, обычно файл с подобными ключами добавляется в `.gitignore`, и либо деплоится специальным образом через настройку now.json либо через файл `.env`. Можно также вручную добавлять константы на сервере типа heroku. Но сейчас нам этот ключ не важен, так что может быть только к финальному деплою сменим.

- Секция рендера тоже разрослась. То, как сейчас написан рендер, противоречит модульности, с которой желательно строить компоненты реакта. Мы можем выделить тупые компоненты `MovieCard` и `Input`, которые будут рендерить стейт из умного `App`, так мы разделим сферы ответственности и нам будет легче дебажить и понимать в чем проблема.

- Пока что я выделяю их как обычные функциональные компоненты, потом, в одной из следующих глав мы отрефакторим их еще с помощью либы styled-components. Она даст нам удобный синтаксис и позволяет класть вместе CSS и JS, что позволяет добиться лучшей модульности и separation of concern, и вообще cutting-edge либа. Выделение таких компонентов — это основа реакта и хорошая практика. В идеале каждый компонент должен знать только и именно то, что ему нужно.

- Также сейчас наш `<div style={{ display: 'flex', flexWrap: 'wrap', justifyContent: 'space-around' }}>` обладает инлайновыми стилями. Ради однообразия с материал-юи и кодстайла мы засунем эти стили в объект withStyles и прокинем пропсами через withStyles HOC.

Сразу покажу конечную файловую структуру и код `App.js`.

<img src="https://raw.githubusercontent.com/Fivapr/Movie-Searcher-Book/master/end-file-structure.png" width=242 alt="Пикча структуры" />

```
import React, { Component } from 'react'
import { withStyles } from '@material-ui/core/styles'
import api from './utils/api'
import MovieCard from './Components/MovieCard'
import Input from './Components/Input'

const styles = {
  container: {
    display: 'flex',
    flexWrap: 'wrap',
    justifyContent: 'space-around'
  }
}

class App extends Component {
  state = { value: '', movies: [] }

  componentDidMount() {
    api('movie/top_rated')
      .then(res => this.setState({ movies: res.data.results }))
      .catch(err => console.log(err))
  }

  onChange = e =>
    this.setState({ value: e.target.value }, () =>
      api(`search/movie`, { query: this.state.value })
        .then(res => this.setState({ movies: res.data.results }))
        .catch(err => console.log(err))
    )

  render() {
    const { classes } = this.props

    return (
      <>
        <Input value={this.state.value} onChange={this.onChange} />
        <div className={classes.container}>
          {this.state.movies.map(movie => (
            <MovieCard key={movie.id} movie={movie} />
          ))}
        </div>
      </>
    )
  }
}

export default withStyles(styles)(App)

```

#### работа с функцией api

В `utils/api` у нас будет одна небольшая функция. Так как мы не будем делать никакие запросы кроме .get, нам не нужен для апи большой класс, обойдемся вот такой функцией (хотя возможно я не прав, и мб получается, что у меня каждый раз будет создаваться новый объект аксиоса? Отревьювьте пожалуйста, нужно ли экспортить класс со статик методом? Или он и так будет один (а то для чего-то на обоих проектах оно было так)).

В этой функции вызывается `axios.get`. Внутри аргументов мы по частям составляем нужный нам урл. Сначала идет `baseURL` — бывший `https://api.themoviedb.org/3/`, потом значащий для апи путь `path` — бывший `search/movie`, дальше идет `?` который разграничивает основной путь и параметры для тмдб апи.

Вторым аргументом аксиоса идет объект опций, одна из которых — специальная опция params. Она конкатенирует к основному пути парамсы в стиле `&api_key=blablabla` и остальные параметры например query поиска `&query=la la land`. Также в объекте опций можно прописывать хэдеры и еще много чего. Но оно нам пока не нужно.

```
import axios from 'axios'
import { baseURL, api_key } from '../config'

export default (path, params) =>
  axios.get(`${baseURL}${path}?`, {
    params: {
      api_key,
      ...params
    }
  })

```

А config.js выглядит так

```
export const baseURL = 'https://api.themoviedb.org/3/'
export const api_key = '59017ce86d5101576f32f47160168519'
```

#### Выделение компонентов

Основной код компонента остался тем же, единственное, что в первой строке мы сразу деструктуризируем объект props, чтобы было удобнее обращаться к вложенным полям movie.

```
const MovieCard = ({ movie, classes }) => (
  ...
```

Также стили в этом компоненте стали модульнее. Внутри этого файла мы имеем только стили, которые относятся к этому компоненту и никаких других. Но их надо немного поменять из-за юишного бага, тк `CardActionArea` не хочет растягиваться и применять свои красивые анимации на всю карту.

Для этого присвоим ей `className={classes.root}`. Где зададим `height: '100%'`, и переделаем дисплей на flex так, чтобы наша пикча и текст карточек отображались нормально сверху вниз. Из-за этого почему-то пикча ужалась в 0 пикселей, но что поделать, зададим ей `width: '100%'`. Также не забываем откопировать все импорты из App.js и обернуть MovieCard в withStyles.

Вообще класс root можно было назвать как-нибудь получше. Но у материал-юи - это стандартное название, поэтому для консистентности беру его.

```
import React from 'react'
import Card from '@material-ui/core/Card'
import CardActionArea from '@material-ui/core/CardActionArea'
import CardContent from '@material-ui/core/CardContent'
import CardMedia from '@material-ui/core/CardMedia'
import Typography from '@material-ui/core/Typography'
import { withStyles } from '@material-ui/core/styles'

const styles = {
  card: {
    maxWidth: 345,
    margin: 10
  },
  media: {
    height: 500,
    width: '100%'
  },
  root: {
    height: '100%',
    display: 'flex',
    justifyContent: 'flex-start',
    alignItems: 'flex-start',
    flexDirection: 'column'
  }
}

const MovieCard = ({ movie, classes }) => (
  <Card className={classes.card} key={movie.id}>
    <CardActionArea className={classes.root}>
      <CardMedia
        className={classes.media}
        image={`https://image.tmdb.org/t/p/w500/${movie.poster_path}`}
        title="Poster"
      />
      <CardContent>
        <Typography gutterBottom variant="h5" component="h2">
          {movie.title}
        </Typography>
        <Typography component="p">{movie.overview}</Typography>
      </CardContent>
    </CardActionArea>
  </Card>
)

export default withStyles(styles)(MovieCard)

```

С `Input.js` ничего сложного не происходит. Просто выделяем компонент и прокидываем ему нужные пропсы для изменения данных в `App.js`.

```
import React from 'react'
import TextField from '@material-ui/core/TextField'

export default ({ value, onChange }) => (
  <TextField
    label="Search for the movie!"
    placeholder="La la land"
    fullWidth
    margin="normal"
    InputLabelProps={{
      shrink: true
    }}
    value={value}
    onChange={onChange}
  />
)

```

Также я вдруг вспомнил, что у нас нет иконки для мувисерчера, свою я нагуглил по запросу icon m. И [здесь](https://www.favicon-generator.org/) сгенерил 32х32 файлик. Дальше просто добавим иконку в паблик фолдер и ссыль на нее в `index.html`.

```
<link rel="shortcut icon" href="%PUBLIC_URL%/favicon.png" />
```

## Деплой с помощью now

Чтобы задеплоить свой невероятный мувисерчер мы глобально установим [now](https://zeit.co/now).

```
npm i -g now
```

Он позволяет задеплоить апп без особой настройки одной командой `now` из рута проекта. Правда вам еще надо будет зарегистрироваться и подтвердить ящик. Но есть проблема, эта конкретная команда задеплоит ваш дев сервер, если вызвать ее вовсе без настроек, он запустит `yarn build` и `yarn start`, это хорошо работает с серверами ноды, но у нас деплой статичный.

[Здесь](https://zeit.co/docs/v1/examples/create-react-app) можно найти пример настройки, который я скопипастил. Просто добавляем файлы оттуда

now.json

```
{
  "type": "static",
  "static": {
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ]
  }
}
```

Dockerfile

```
# Use Node.js version 10
FROM mhart/alpine-node:10

# Set the working directory
WORKDIR /usr/src

# Copy package manager files to the working directory and run install
COPY package.json yarn.lock ./
RUN yarn install

# Copy all files to the working directory
COPY . .

# Build the app and move the resulting build to the `/public` directory
RUN yarn build
RUN mv ./build /public
```

.dockerignore

```
*
!src
!public
!package.json
!yarn.lock
```

Я не шарю в докере, не спрашивайте меня подробности. Но фактически все, что он делает, — это кладет наш статичный билд в /public так, чтобы index.js и любые запросы файлов могли попасть в него и забрать, что надо строго в соответствии с путем после now.sh/. Рассматривайте это просто как папку с файлами. Теперь деплоим!

```
now
```

Этот процесс займет около минуты, и now вам выдаст какое-то всратое название места, куда вы задеплоили, уровня moviesearcher-scnsedikmv.now.sh. Чтобы его сменить, нужно выполнить команду `now alias`

```
now alias moviesearcher-scnsedikmv.now.sh moviesearcher
```

По ее завершению ваш серчер будет доступен по адресу moviesearcher.now.sh. А чтобы не вводить эту команду при каждом деплое мы добавим файл `now.json` с настройками деплоя. В него мы добавим одну строку

```
"alias": "moviesearcher"
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

Ура! Мувисерчер теперь в интернете!

Итоговый код первой главы можно найти в [этом коммите](https://github.com/Fivapr/Movie-Searcher-App/commit/28e65077e0728bbb654659c178f21a059001c2a0)
