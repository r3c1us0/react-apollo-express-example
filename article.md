# How to Create application using React, Apollo graphql and Express

## Step 1. Create server using Express and Apollo graphql server

### 1.1 Initialize the application

Для начала нам нужно создать папку, перейти в неё, а затем инициализировать в ней наше приложение при помощи npm.

```bash
mkdir react-apollo-express-example
cd react-apollo-express-example
npm init
```

### 1.2 Install server dependencies

После инициализации нашего приложения нам необходимо установить зависимости для создания сервера.

```bash
npm install express apollo-server-express graphql --save
```

### 1.3 Create graphql schema for the Posts module

Далее создаем папку `server`. Следующим шагом будет создание graphql схемы для описания types, queries and mutations которые будут нужны для работы с Posts module. Для этого мы создаем файл `graphqlSchema.js` в папке `server` и добавляем в него следующий код.

```javascript
const { gql } = require('apollo-server-express');

// Construct a schema, using GraphQL schema language
const typeDefs = gql`
  type Post {
    title: String,
    content: String
  },
  type Query {
    posts: [Post]
  },
  type Mutation {
    addPost(title: String!, content: String!): Post,
  }
`;

module.exports = typeDefs;
```

### 1.4 Implement graphql Query and Mutation resolvers

После создания graphql схемы мы создаём resolvers для обработки Queries and Mutations. Создаем файл `resolvers.js` в папке `server`. Далее создаем `Query` которое будет возвращать список постов. Поскольку в данном приложении не используеться база данных, то мы создадим массив в котором будет хранится список наших постов.
После этого, мы добавляем `Mutation`, которая добавляет новый пост в наш список и затем возвращает наш пост, если он был успешно добавлен.

```javascript

// Dummy data
const posts = [
  { title: 'Post Title1', content: 'Lorem Ipsum is simply dummy text of the printing and typesetting industry.' },
  { title: 'Post Title2', content: "Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book." },
  { title: 'Post Title3', content: 'Contrary to popular belief, Lorem Ipsum is not simply random text. ' }
];

// Provide resolver functions for your schema fields
const resolvers = {
  Query: {
    // Query which returns posts list
    posts: () => posts,
  },
  Mutation: {
    /* Mutation which provides functionality for adding post to the posts list 
    * and return post after successfully adding to list
    */
    addPost: (_, post) => {
      posts.push(post);
      return post;
    }
  }
};

module.exports = resolvers;
```

### 1.5 Server setup

Саздаем файл `server.js` и добавляем в него код для запуска сервера. Так же мы подключаем `graphql schema` и `resolvers` в наш файл и передаем их в `ApolloServer`

```javascript
const express = require('express');
const { ApolloServer } = require('apollo-server-express');

const typeDefs = require('./graphqlSchema');
const resolvers = require('./resolvers');

// Create Apollo server
const server = new ApolloServer({ typeDefs, resolvers });

// Create Express app
const app = express();

server.applyMiddleware({ app });

// Listen server
app.listen({ port: 3000 }, () =>
  console.log(`🚀 Server ready at http://localhost:3000${server.graphqlPath}`)
);
```
### 1.6 Create command for starting sever

В первую очередь, мы должы установить следующий пакет для горячей перезагрузки серверного кода. Эта зависимость должа быть установленна глобально

```bash
npm install nodemon -g
```

После установки `nodemon` добавляем следующий код в секцию `scripts` в файле `package.json`.

```json
"server": "nodemon ./server/server.js"
```

## Step 2. Create client using React and Apollo client

### 2.1 Install client dependencies for configuring webpack

Для начала нам нужно установить зависимости для того, что бы настроить `webpack` для клиентской стороны.

```bash
npm install webpack webpack-cli webpack-dev-server html-webpack-plugin css-loader style-loader --save-dev
```

### 2.2 Configure webpack

Создаем в корне проекта файл `webpack.config.js` и добавляем туда следующий код

```javascript
const HtmlWebPackPlugin = require("html-webpack-plugin");
const path = require('path');

const htmlPlugin = new HtmlWebPackPlugin({
  template: "./client/src/index.html",
  filename: "./index.html"
});

module.exports = {
  entry: './client/src',
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      },
      {
        test: /\.css$/,
        use: ["style-loader", "css-loader"]
      }
    ]
  },
  plugins: [htmlPlugin]
};
```

### 2.3 Install babel dependencies

Добавляем зависимости для дальнейшей конфигурации `babel`.

```bash
npm install babel-core babel-loader babel-preset-env babel-preset-react --save-dev
```

### 2.4 Babel configuration

Создаем в корне проекта файл `.babelrc` и добавляем следующий код

```
{
  "presets": [
    "env",
    "react"
  ]
}
```

### 2.5 Install dependencies for React

Устанавливаем следующие зависимости для клиентской стороны

```bash
npm install react react-dom --save
```

Так же нам я решил установить UI toolkit для приложения. Я выбрал `reactstrap`, но вы можете выбрать любой более подходящий для вас. Устанавливаем пакеты `reactstrap` и `bootstrap`.

```bash
npm install reactstrap bootstrap --save
```

После этого устанавливаем зависимости для `Apollo client`.

```bash
npm install apollo-boost react-apollo --save
```

### 2.6 Initialize React client

После установки всех зависимостей приступим к разработке нашего React клиента.

Для начала в корне проекта создаем папку `client` и в ней папку `src`. Далее создадим папку в которой в будующем будем хранить конфигурационные файлы и назовем её `settings`. После этого создаем папку `modules`. Эта папка будет содержать модули вашего приложения такие как: `user, products, auth and etc.`. В нашем примере мы создадим модуль `posts`.

В папке `src` создаем файл `index.html` и добавляем туда следующий код

```html
<!DOCTYPE html>
<html lang="en">

  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>React Apollo Express Example</title>
  </head>

  <body>
    <section id="root"></section>
  </body>

</html>
```

Теперь создаем файл `App.js` в котором будет инициализироваться `Apollo client`. Так же, в дальнейшем, вы можете добавить в этот файл роутинг.

На данном этапе мы выведем в этом файле Hello world! Добавьте следующий код в файл `App.js`.

```javascript
import React, { Component } from 'react'

class App extends Component {
  render() {
    return (
      <div>Hello world!</div>
    )
  }
}

export default App;
```

Теперь создадим файл, который будет точкой входа в наше клиентское веб приложение.
В папке `client` создадим файл `index.js` и добавим туда этот код

```javascript
import React from "react";
import ReactDOM from "react-dom";
import App from './App';

ReactDOM.render(<App />, document.getElementById("root"));
```

Теперь нам нужно создать команду которая будет запускать клиентское приложение. Для этого добавляем следующую команду в раздел `scripts` в файле `package.json`

```json
"client": "webpack-dev-server --mode development --open"
```

### 2.7 Initialize Apollo client

Теперь нам нужно создать `Apollo client`. Для этого в папке `settings` создадим файл `createApolloClient.js` и добавим туда код для создания `Apollo client`.

```javascript
import ApolloClient from "apollo-boost";

const client = new ApolloClient({
  uri: "http://localhost:3000/graphql"
});

export default client;
```

Теперь нам нужно добавить `ApolloProvider` и передать в него наш `ApolloClient`.
В первую очередь нам нужно импортировать `ApolloClient` и `ApolloProvider` в файле `App.js`.

```javascript
import { ApolloProvider } from 'react-apollo';

import apolloClient from './settings/createApolloClient';
```

Теперь нам нужно обернуть наше приложение в `ApolloProvider` что бы иметь достпук к данным `AppoloClient`.

После этого файл `App.js` должен содержать следующий код.

```javascript
import React, { Component } from 'react'

import { ApolloProvider } from 'react-apollo';

import apolloClient from './settings/createApolloClient';


class App extends Component {
  render() {
    return (
      <ApolloProvider client={apolloClient}>
        Hello world!
      </ApolloProvider>
    )
  }
}

export default App;
```