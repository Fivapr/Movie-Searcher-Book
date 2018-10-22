---
title: MovieSearcher Book
seoTitle: moviesearcher app
seoDescription: moviesearcher app for frontend thread
isFree: true
---

## Введение или что такое мувисерчер для чего эта книга?

Мувисерчер — тестовое SPA после которого каждый анон может идти покорять эйчарок. С его помощью вы сможете найти работу до 50к/наносек в своем миллионнике или до 80к/наносек в ДСе. Сегодня и только сегодня первая вводная глава бесплатна — это тизер обширного туториала по миру фронтэнда, цена которого будет всего 10$. 10$ сегодня, 1000$ заработка уже через месяц!

#### Курс овервью и стек

Наш мувисерчер будет построен с помощью либ React, Redux, Redux-Saga, ImmutableJS, Reselect, Material-UI, Redux-symbiote и нескольких других полезных либ. Функционал покроет обычный поиск, страничку фильма, поиск по фильтрам и добавление любимых фильмов в локалстораж. Сначала мы напишем простейший серчер по строке на реакте, потом перенесем стейт в редакс и поочередно будем включать и рефакторить наш код с множеством новых либ.

## React

Прежде, чем начинать пилить серчер, тебя, мой уважаемый анон, может заинтересовать [дока реакта](https://reactjs.org/docs/getting-started.html) и, откуда бы вы их не взяли, знания жаваскрипта и ес6 фич типа классов и стрелочных функций.

А теперь начнем.

В общем где-то здесь будет код серчера и объяснения. Приглашаю добровольцев для помощи в написании этой невероятной книги, а сегодня я устал.

```
import React from "react";
import { render } from "react-dom";
import { BrowserRouter } from "react-router-dom";
import { createStore, applyMiddleware, compose } from "redux";
import { Provider } from "react-redux";
import { rootReducer } from "./RootReducer";
import { composeWithDevTools } from "redux-devtools-extension";
import createSagaMiddleware from "redux-saga";
import rootSaga from "./RootSaga";
import WebFontLoader from "webfontloader";
import "./index.css";
import App from "./App";

WebFontLoader.load({
  google: {
    families: ["Roboto:300,400,500,700", "Material Icons"]
  }
});

const sagaMiddleware = createSagaMiddleware();

const initialState = {};

const store = createStore(
  rootReducer,
  initialState,
  composeWithDevTools(applyMiddleware(sagaMiddleware))
);

sagaMiddleware.run(rootSaga);

render(
  <Provider store={store}>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </Provider>,
  document.getElementById("root")
);

// ========================================

лал
```
