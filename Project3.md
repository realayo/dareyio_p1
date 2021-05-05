# A To-Do application on MERN Web Stack

### This simple application makes use of the following web stack
1. **MongoDB** - NoSQL that stores application data inform of documents
1. **ExpressJs** - A server side web app framework for Node.js
1. **ReactJs** - A JS library for building frontend UI or UI components
1. **Node.js** - A JS runtime environment that executes javascript codes.

## Backend configuration

1. After spinning up on EC2 instance on AWS and setting up the necessary security groups(SSH and HTTP), we login to the instance using our perm key and do a ``sudo apt update`` after that, we update ubuntun using ``sudo apt upgrade``
1. We download node.js into our server ![](https://user-images.githubusercontent.com/18899718/115121160-cda48280-9f76-11eb-8fd5-dd156d177579.png)
1. Install nodejs and npm(package manager for node) using ``sudo apt-get install -y nodejs``
Verify node and npm installation.![](https://user-images.githubusercontent.com/18899718/115121263-4c99bb00-9f77-11eb-8f6b-09bbae5ac7b8.png)
1. We create a new directory for our Todo project using ``mkdir Todo``
1. Change direcotry to Todo ``cd Todo``
1. In **Todo** directory, we run ``npm init`` to initialise our project to create a `package.json` file.![](https://user-images.githubusercontent.com/18899718/115121477-8f0fc780-9f78-11eb-93be-c1c233e50a07.png)

### Install ExpressJs

1. We run ``npm install express``
We create an **index.js** file using ``touch index.js``
1. Next we install a module that creates environmental variable 
``npm install dotenv``
1. we open the index.js file we created using ``vi index.js`` and paste the following codes inside. ![](https://user-images.githubusercontent.com/18899718/115122999-989d2d80-9f80-11eb-909f-c20604255009.png)

1. We run ``node index.js`` to be sure our server is running. 
![](https://user-images.githubusercontent.com/18899718/115123259-c8006a00-9f81-11eb-876a-eaa6506e2134.png)
1. As we can see it's running port 5000, we need to go to our Security group in AWS console and enable port 5000
1. Next we open our browser and access our server IP followed by port 5000
![](https://user-images.githubusercontent.com/18899718/115123389-6bea1580-9f82-11eb-9374-88b5485ad30d.png)


### Routes

1. Our app is should be able to do the following things:
    * Create a new task
    * Display a list of tasks
    * Delete completed tasks
1. To make these happen, we need to create ``routes`` for each task, which would essentially map out the various endpoints that our app would depend on 

1. To achieve this, in the *Todo* folder, we make a new directory ``mkdir routes``.
1. Change direcotry to ``routes`` using ``cd routes``
1. We create a new file ``api.js`` using ``touch api.js``
1. ``vim api.js`` then copy the following code inside.
```
const express = require ('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

})

module.exports = router;
```
7. Save and exit



### Models
1. To create a schema and a model, we install ``mongoose``, a package maanger for node.js that makes workign with mongodb easier.
1. First we change directory back to *Todo* folder, then install Mongoose using ```npm install mongoose``` ![](https://user-images.githubusercontent.com/18899718/115128749-a4e7b180-9fa5-11eb-91ce-4ba35514d246.png)
1. Create a new folder `models` using ```mkdir models```
1. Change directory to ``models`` and create a file ``todo.js``
1. open ``todo.js`` and add the following code ![](https://user-images.githubusercontent.com/18899718/115128811-22abbd00-9fa6-11eb-8ec1-e1c43cdd85be.png)
1. Next up, we update our ``api.js`` file in ``routes`` directory so it can make use of our new models. 
1. ``cd routes``, ``vim api.js``, then delete the content and update with the content below
![](https://user-images.githubusercontent.com/18899718/115128896-a9f93080-9fa6-11eb-8636-ea553dd5f513.png)
1. Save and exit

### MongoDB 
1. We create an account on mongodb.com.
1. Under network access, we allow access to our DB from anywhere(which is not a good practice)
1. Under ``clusters``, we go to ``collections``, where we create our own database.
1. To get our connection string, We go back to ``clusters`` again and click on ``connect``, then click on ``connect yoru application``, there we would have our connection string ![connection string](https://user-images.githubusercontent.com/18899718/115129215-98fdee80-9fa9-11eb-93ad-617cbd2eabe4.png)
1. Now, we need to create an environmental file that have our database connection string. In the **Todo** directory, type this
``` touch .env``, after that, ``vim .env``
then we paste our connection string from above inside. using the following syntax 
```
DB = 'mongodb+srv://<username>:<password>@<network-address>/<dbname>?retryWrites=true&w=majority'
```
making sure to replace every parameter with the right ones.

6. We have to update our index.js file to reflect our new ``.env`` variable.
1. Do a ``vim index.js`` and replace the content with what we have below ![](https://user-images.githubusercontent.com/18899718/115129374-e3cc3600-9faa-11eb-9887-ee9092282ea5.png)

1. Now, we start the server by running ``node index.js``
![](https://user-images.githubusercontent.com/18899718/115129401-27bf3b00-9fab-11eb-879a-4cab796a103a.png)


### Test backend code using RESTful API

The test would be done inside postman. 
1. Under ``headers``, we select ``content-Type`` under ``KEY`` and ``application/json`` under ```VALUE``  ![](https://user-images.githubusercontent.com/18899718/115129454-ad42eb00-9fab-11eb-96da-532bf5d3012c.png)

1. We input some data in the ``raw`` field and see if we get some output ![](https://user-images.githubusercontent.com/18899718/115129475-ed09d280-9fab-11eb-9a4f-56968dfdca43.png)
1. We create a ``GET`` request to retrieve the data we ``POST`
![](https://user-images.githubusercontent.com/18899718/115129549-9c46a980-9fac-11eb-9934-977aae3c7f8a.png)


### Frontend Creation

We have to install a couple of dependencies.
1. concurrently, we can install this using  ```npm install concurrently --save-dev```
1. Next is nodemon which is used to run and monitor the server ``npm install nodemon --save-dev``
1. In our **Todo** folder, we open ``package.json`` and replace the ``"scripts":{`` part witht he following code
```
"scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
},
```
4. Change directory to ``clients``
1. Open ``package.json`` and add the following key value pair ```proxy": "http://localhost:5000".```. Save and exit
1. Under the instance security group, add a custom TCP port 3000 so we'll be able to access our frontend
![](https://user-images.githubusercontent.com/18899718/115129761-8fc35080-9fae-11eb-8dd7-0b77e829f7b8.png)


### Creating React Components
1. ``cd client``
1. To go ``src`` folder ``cd src``
1. We create a folder called components ``mkdrir components``
1. we created 3 files ruuning this command ```touch Input.js ListTodo.js Todo.js```
1. Open ``input.js``
copy and paste the following code
```
import React, { Component } from 'react';
import axios from 'axios';

class Input extends Component {

state = {
action: ""
}

addTodo = () => {
const task = {action: this.state.action}

    if(task.action && task.action.length > 0){
      axios.post('/api/todos', task)
        .then(res => {
          if(res.data){
            this.props.getTodos();
            this.setState({action: ""})
          }
        })
        .catch(err => console.log(err))
    }else {
      console.log('input field required')
    }

}

handleChange = (e) => {
this.setState({
action: e.target.value
})
}

render() {
let { action } = this.state;
return (
<div>
<input type="text" onChange={this.handleChange} value={action} />
<button onClick={this.addTodo}>add todo</button>
</div>
)
}
}

export default Input
```
6. Next we install ``Axios`` in the ``clients`` directory.
``npm install axios``
1. We go back to components directory by doing ``cd src/components``
1. We open ``ListTodo.js`` using ``vim ListTodo.js``, then copy and past the following code inside.
```
import React from 'react';

const ListTodo = ({ todos, deleteTodo }) => {

return (
<ul>
{
todos &&
todos.length > 0 ?
(
todos.map(todo => {
return (
<li key={todo._id} onClick={() => deleteTodo(todo._id)}>{todo.action}</li>
)
})
)
:
(
<li>No todo(s) left</li>
)
}
</ul>
)
}

export default ListTodo
```

9. For ``Todo.js``, we paste the following code inside
```
import React, {Component} from 'react';
import axios from 'axios';

import Input from './Input';
import ListTodo from './ListTodo';

class Todo extends Component {

state = {
todos: []
}

componentDidMount(){
this.getTodos();
}

getTodos = () => {
axios.get('/api/todos')
.then(res => {
if(res.data){
this.setState({
todos: res.data
})
}
})
.catch(err => console.log(err))
}

deleteTodo = (id) => {

    axios.delete(`/api/todos/${id}`)
      .then(res => {
        if(res.data){
          this.getTodos()
        }
      })
      .catch(err => console.log(err))

}

render() {
let { todos } = this.state;

    return(
      <div>
        <h1>My Todo(s)</h1>
        <Input getTodos={this.getTodos}/>
        <ListTodo todos={todos} deleteTodo={this.deleteTodo}/>
      </div>
    )

}
}

export default Todo;
```
10. We go back to our ``src`` folder.
1. Open ``app.js`` and paste the following code
```
import React from 'react';

import Todo from './components/Todo';
import './App.css';

const App = () => {
return (
<div className="App">
<Todo />
</div>
);
}

export default App;
```
save and exit

12. Next, ``vim App.css``, paste the following 
```
.App {
text-align: center;
font-size: calc(10px + 2vmin);
width: 60%;
margin-left: auto;
margin-right: auto;
}

input {
height: 40px;
width: 50%;
border: none;
border-bottom: 2px #101113 solid;
background: none;
font-size: 1.5rem;
color: #787a80;
}

input:focus {
outline: none;
}

button {
width: 25%;
height: 45px;
border: none;
margin-left: 10px;
font-size: 25px;
background: #101113;
border-radius: 5px;
color: #787a80;
cursor: pointer;
}

button:focus {
outline: none;
}

ul {
list-style: none;
text-align: left;
padding: 15px;
background: #171a1f;
border-radius: 5px;
}

li {
padding: 15px;
font-size: 1.5rem;
margin-bottom: 15px;
background: #282c34;
border-radius: 5px;
overflow-wrap: break-word;
cursor: pointer;
}

@media only screen and (min-width: 300px) {
.App {
width: 80%;
}

input {
width: 100%
}

button {
width: 100%;
margin-top: 15px;
margin-left: 0;
}
}

@media only screen and (min-width: 640px) {
.App {
width: 60%;
}

input {
width: 50%;
}

button {
width: 30%;
margin-left: 10px;
margin-top: 0;
}
}
```
save and exit.

13. Open ``index.css``, past the following code
```
body {
margin: 0;
padding: 0;
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen",
"Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue",
sans-serif;
-webkit-font-smoothing: antialiased;
-moz-osx-font-smoothing: grayscale;
box-sizing: border-box;
background-color: #282c34;
color: #787a80;
}

code {
font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New",
monospace;
}
```
14. We go back to ``Todo`` directory by running ``cd ../..``
1. In the directory, we run ``npm run dev``
![](https://user-images.githubusercontent.com/18899718/115130126-702e2700-9fb2-11eb-8700-8b16ab7b418a.png)

We can go the web browser and check out our running app.![](https://user-images.githubusercontent.com/18899718/115130144-9227a980-9fb2-11eb-8b7a-773615c3d085.png)