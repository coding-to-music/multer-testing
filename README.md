# multer-testing

This is a guide on how to use the `multer` library with `express`.

We start with setting up a basic form for uploading a file and other text information(optional).

I used `React` for this tutorial but the same result can be easily made with any other framework or with plain JavaScript.

## Quick Start

clone the repo




Start the front end
```java
cd Client

# install gatsby if needed
apt update -y && apt upgrade -y

# become root if needed
sudo apt update -y && sudo apt upgrade -y

# verify versions
node -v && npm -v

# install gatsby
npm install -g gatsby-cli
```

run gatsby locally
```java
gatsby develop
```java

Output
```java
success open and validate gatsby-configs - 0.060s
success load plugins - 0.050s
success onPreInit - 0.033s
success initialize cache - 0.008s
success copy gatsby files - 0.061s
success onPreBootstrap - 0.025s
success createSchemaCustomization - 0.005s
success Checking for changed pages - 0.007s
success source and transform nodes - 0.062s
success building schema - 0.249s
info Total nodes: 18, SitePage nodes: 1 (use --verbose for breakdown)
success createPages - 0.014s
success Checking for changed pages - 0.001s
success createPagesStatefully - 0.054s
success update schema - 0.032s
success write out redirect data - 0.009s
success onPostBootstrap - 0.002s
info bootstrap finished - 3.654s
success onPreExtractQueries - 0.002s
success extract queries from components - 0.130s
success write out requires - 0.010s
success run page queries - 0.032s - 1/1 30.84/s
warn Browserslist: caniuse-lite is outdated. Please run:
npx browserslist@latest --update-db

Why you should do it regularly:
https://github.com/browserslist/browserslist#browsers-data-updating
⠀
You can now view client in the browser.
⠀
  http://localhost:8000/
⠀
View GraphiQL, an in-browser IDE, to explore your site's data and schema
⠀
  http://localhost:8000/___graphql
⠀
Note that the development build is not optimized.
To create a production build, use gatsby build
⠀
success Building development bundle - 12.300s
```
![image](https://user-images.githubusercontent.com/3156358/140828616-ad6f9848-93a8-435a-bc70-0b37baa2fd78.png)


Set up the back end
```java
cd server



## Creating a basic React project

In order to start we need a working React project. Simply run this command in your terminal to set up a basic React project.

```shell
npx create-react-app <project_name>
```
**Note:** Replace `<project_name>` with whatever you want to call your project.
To check that everything is working run `cd <project_name>` and `npm start`. You should see a boilerplate React app in your browser.

## Creating the form for uploading

We will make a form that will be used to upload files and a title for that file.

App.js
```jsx
import React from 'react';

const App = () => {
    return (
        <form>
            <input type="text" name="text" />
            <input type="file" name="file" />
            <input type="submit" value="Submit" />
        </form>
);
    };

export default App;
```

## Now we will set up a server using multer.js

**Note**: In order to start run the following command in a folder on the same level as the React project.

1. First initialize a node project in the folder for the server.

```shell
npm init -y
```

2. Then install `express` and `multer` using the following command.

```shell
npm i -D express multer cors body-parser
```

3. In your `package.json` we need to change some things

### Add the following to your `scripts`

```json
"scripts": {
    "start": "node index.js"
}
```

### Also add type setting

```json
"type": "module"
```

4. Make a `index.js` file for the server

```js
import express from 'express';
import bodyparser from 'body-parser';
import cors from 'cors';

const app = express();

app.get('/posts', (req, res) => {});

app.post('/submit', (req, res) => {});

app.listen(3030, () => console.log('server listening on port 3030'));
```

### We also need to set up some middleware

```js
import express from 'express';
import bodyparser from 'body-parser';
import cors from 'cors';

const app = express();

app.use(cors());
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());

app.use('/uploads', express.static('./uploads'));

app.get('/posts', (req, res) => {});

app.post('/submit', (req, res) => {});

app.listen(3030, () => console.log('server listening on port 3030'));
```

5. Now let's prepare multer

```js
import express from 'express';
import bodyparser from 'body-parser';
import cors from 'cors';
import multer from 'multer';

const app = express();

app.use(cors());
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());

var storage = multer.diskStorage({
  destination: function (req, file, cb) {
    cb(null, './uploads');
  },
  filename: function (req, file, cb) {
    cb(null, file.fieldname + '-' + Date.now() + '.jpg');
  },
});

var upload = multer({ storage: storage });

app.use('/uploads', express.static('./uploads'));

app.get('/posts', (req, res) => {});

app.post('/submit', upload.single('file'), (req, res) => {});

app.listen(3030, () => console.log('server listening on port 3030'));
```

6. Now make a `uploads` file right next to the `index.js`

7. Let's set up MongoDB

Run this command

```shell
npm i -D mongoose
```

index.js
```js
import express from 'express';
import bodyparser from 'body-parser';
import cors from 'cors';
import multer from 'multer';
import mongoose from 'mongoose';

const app = express();

app.use(cors());
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());

var storage = multer.diskStorage({
  destination: function (req, file, cb) {
    cb(null, './uploads');
  },
  filename: function (req, file, cb) {
    cb(null, file.fieldname + '-' + Date.now() + '.jpg');
  },
});

var upload = multer({ storage: storage });

mongoose
  .connect('mongodb://localhost:27017/multer-test', {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  })
  .then(res => console.log('DB connected'))
  .catch(err => console.error(err));


app.use('/uploads', express.static('./uploads'));

app.get('/posts', (req, res) => {});

app.post('/submit', upload.single('file'), (req, res) => {});

app.listen(3030, () => console.log('server listening on port 3030'));

```

### Now we will create a model for the database

models/Test.js
```js
import mongoose from 'mongoose';

const test_schema = new mongoose.Schema({
  file_path: {
    type: String,
    required: true,
  },
  description: {
    type: String,
    required: true,
  },
});

export default mongoose.model('Test', test_schema);
```

And after that we can use the database

index.js
```js
import express from 'express';
import bodyparser from 'body-parser';
import cors from 'cors';
import multer from 'multer';
import mongoose from 'mongoose';
import Test from './models/Test.js';

const app = express();

app.use(cors());
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());

var storage = multer.diskStorage({
  destination: function (req, file, cb) {
    cb(null, './uploads');
  },
  filename: function (req, file, cb) {
    cb(null, file.fieldname + '-' + Date.now() + '.jpg');
  },
});

var upload = multer({ storage: storage });

mongoose
  .connect('mongodb://localhost:27017/multer-test', {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  })
  .then(res => console.log('DB connected'))
  .catch(err => console.error(err));


app.use('/uploads', express.static('./uploads'));

app.get('/posts', (req, res) => {
    Test.find({})
        .then(response => res.json(response))
        .catch(err => console.error(err));
});

app.post('/submit', upload.single('file'), (req, res) => {
    const data = new Test({ description: req.body.text, file_path: req.file.path });
    data.save()
        .then(response => console.log(response))
        .catch(err => console.error(err));
});

app.listen(3030, () => console.log('server listening on port 3030'));
```
**Note:** This completes our server.

## Now we will make a request from the server in order to upload a file

Back in our `React` project we run:

```shell
npm i -D axios
```

src/App.js
```jsx
import React, { useRef } from 'react';
import axios from 'axios';

const App = () => {
    const formRef = useRef(null);
    const submit_file = e => {
        e.preventDefault();

        const form_data = new FormData(formRef.current);

        axios({
            url: 'http://localhost:3030/submit',
            method: 'post',
            headers: { 'Content-Type': 'multipart/form-data' },
            data: form_data
        })
            .then(res => console.log(res))
            .catch(err => console.error(err));
    };

    return (
        <form onSubmit={submit_file} ref={formRef}>
            <input type="text" name="text" />
            <input type="file" name="file" />
            <input type="submit" value="Submit" />
        </form>
);
    };

export default App;
```

### Now we can upload files and save their path to the database

Also if we want access to our files and the data related to them we can make another `axios` request to `http://localhost:3030/posts`.

src/App.js
```jsx
import React, { useRef, useState, useEffect } from 'react';
import axios from 'axios';

const App = () => {
    const formRef = useRef(null);
    const [data, setData] = useState([]);

    useEffect(() => {
        axios.get('http://localhost:3030/posts')
            .then(res => setData(res.data))
            .catch(err => console.error(err));
    }, []);

    const submit_file = e => {
        e.preventDefault();

        const form_data = new FormData(formRef.current);

        axios({
            url: 'http://localhost:3030/submit',
            method: 'post',
            headers: { 'Content-Type': 'multipart/form-data' },
            data: form_data
        })
            .then(res => console.log(res))
            .catch(err => console.error(err));
    };

    return (
        <form onSubmit={submit_file} ref={formRef}>
            <input type="text" name="text" />
            <input type="file" name="file" />
            <input type="submit" value="Submit" />
        </form>
);
    };

export default App;
```

Now we have acces to the file path and text within our `data` array.

src/App.js
```jsx
import React, { useRef, useState, useEffect } from 'react';
import axios from 'axios';

const App = () => {
    const formRef = useRef(null);
    const [data, setData] = useState([]);

    useEffect(() => {
        axios.get('http://localhost:3030/posts')
            .then(res => setData(res.data))
            .catch(err => console.error(err));
    }, []);

    const submit_file = e => {
        e.preventDefault();

        const form_data = new FormData(formRef.current);

        axios({
            url: 'http://localhost:3030/submit',
            method: 'post',
            headers: { 'Content-Type': 'multipart/form-data' },
            data: form_data
        })
            .then(res => console.log(res))
            .catch(err => console.error(err));
    };

    return (
        <>
            <form onSubmit={submit_file} ref={formRef}>
                <input type="text" name="text" />
                <input type="file" name="file" />
                <input type="submit" value="Submit" />
            </form>
            <div>
            {data.map(el => (
                <div key={el._id}>
                    <h2>{ el.description }</h2>
                    <img src={`http://localhost:3030/${el.file_path.replace('\\', '/')}`} />
                </div>
            ))}
            </div>
        </>
    );
};

export default App;
```

This is it now you can upload files through a form.
