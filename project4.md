# MEAN Stack Deployment to Ubuntu on AWS

## MEAN Stack has the following components:
1. **MongoDB** (Document database) - Stores and allows to retrieve data.
1. **Express** (Back-end application framework) - Makes requests to Database for Reads and Writes.
1. **Angular** (Front-end application framework) - Handles Client and Server Requests.
1. **Node.js** (JavaScript runtime environment) - Accepts requests and displays results to end user.

## The Task
We're going to implement a simple book register


## Install Node.js
1. Update ubuntu `` sudo apt update``
1. Upgrade Ubuntu `` sudo apt upgrade``
1. Install node.js ``sudo apt install -y nodejs``

## Install MongoDB
1. ``sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6`` ![](https://user-images.githubusercontent.com/18899718/117191896-b8b55500-ada6-11eb-9175-5c3c83a4ea4e.png)
1. echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list 

1. install MongoDB ``sudo apt install -y mongodb``
1. start the server ``sudo service mongodb start``
1. Verify the service is running ![](https://user-images.githubusercontent.com/18899718/117192159-029e3b00-ada7-11eb-99c3-cb7e8f59707c.png)
1. Install node package manager
``sudo apt install -y npm``
1. Install body-parser package which help us process JSON files passed in requests to the server.
``sudo npm install body-parser``
1. We create a folder called Books in CD into it
``mkdir Books && cd Books``
1. In the Books directory, we initialise npm
``npm init``
![](https://user-images.githubusercontent.com/18899718/117193113-18603000-ada8-11eb-966b-6c007528caa6.png)
1. we add a file called server.js
``touch server.js``
1. We edit the server.js file
``vi server.js``
Copy and paste the following code inside:
```
var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./apps/routes')(app);
app.set('port', 3300);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});
```
## Install Express.js and setup routes

1. ``sudo npm install express mongoose``
1. In **Books** folder, we create a folder named **apps**
``mkdir apps && cd apps``
1. In the apps folder, we create a file named routes.js
``touch route.js``
We edit route.js ``vim route.js`` and paste the following code inside:
```
var Book = require('./models/book');
module.exports = function(app) {
  app.get('/book', function(req, res) {
    Book.find({}, function(err, result) {
      if ( err ) throw err;
      res.json(result);
    });
  }); 
  app.post('/book', function(req, res) {
    var book = new Book( {
      name:req.body.name,
      isbn:req.body.isbn,
      author:req.body.author,
      pages:req.body.pages
    });
    book.save(function(err, result) {
      if ( err ) throw err;
      res.json( {
        message:"Successfully added book",
        book:result
      });
    });
  });
  app.delete("/book/:isbn", function(req, res) {
    Book.findOneAndRemove(req.query, function(err, result) {
      if ( err ) throw err;
      res.json( {
        message: "Successfully deleted the book",
        book: result
      });
    });
  });
  var path = require('path');
  app.get('*', function(req, res) {
    res.sendfile(path.join(__dirname + '/public', 'index.html'));
  });
};
```
4. In the **apps** folder, we make another folder called **models** 
``mkdir models && cd models``
In **models** folder, we create **books.js** in edit it. 
``touch book.js``
1. we do ``vim book.js`` and past the folloing code inside:
```
var mongoose = require('mongoose');
var dbHost = 'mongodb://localhost:27017/test';
mongoose.connect(dbHost);
mongoose.connection;
mongoose.set('debug', true);
var bookSchema = mongoose.Schema( {
  name: String,
  isbn: {type: String, index: true},
  author: String,
  pages: Number
});
var Book = mongoose.model('Book', bookSchema);
module.exports = mongoose.model('Book', bookSchema);

```

## Access the routes with AngularJS

We use AngularJS to connect our web page with Express and perform actions on our book register.

1. We change directory to **Books**  create a folder named **public**, then change into **public* folder
``mkdir public && cd public``
1. create a file named **script.js**
``touch script.js`` and edit it ``vim script.js``
and paste the following code inside:
```
var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope, $http) {
  $http( {
    method: 'GET',
    url: '/book'
  }).then(function successCallback(response) {
    $scope.books = response.data;
  }, function errorCallback(response) {
    console.log('Error: ' + response);
  });
  $scope.del_book = function(book) {
    $http( {
      method: 'DELETE',
      url: '/book/:isbn',
      params: {'isbn': book.isbn}
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
  $scope.add_book = function() {
    var body = '{ "name": "' + $scope.Name + 
    '", "isbn": "' + $scope.Isbn +
    '", "author": "' + $scope.Author + 
    '", "pages": "' + $scope.Pages + '" }';
    $http({
      method: 'POST',
      url: '/book',
      data: body
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
});
```
3. Create a file named **index.html** inside the **public folder**
``touch index.html``; ``vim index.html`` and paste the following code inside:
```
<!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
    <script src="script.js"></script>
  </head>
  <body>
    <div>
      <table>
        <tr>
          <td>Name:</td>
          <td><input type="text" ng-model="Name"></td>
        </tr>
        <tr>
          <td>Isbn:</td>
          <td><input type="text" ng-model="Isbn"></td>
        </tr>
        <tr>
          <td>Author:</td>
          <td><input type="text" ng-model="Author"></td>
        </tr>
        <tr>
          <td>Pages:</td>
          <td><input type="number" ng-model="Pages"></td>
        </tr>
      </table>
      <button ng-click="add_book()">Add</button>
    </div>
    <hr>
    <div>
      <table>
        <tr>
          <th>Name</th>
          <th>Isbn</th>
          <th>Author</th>
          <th>Pages</th>

        </tr>
        <tr ng-repeat="book in books">
          <td>{{book.name}}</td>
          <td>{{book.isbn}}</td>
          <td>{{book.author}}</td>
          <td>{{book.pages}}</td>

          <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
        </tr>
      </table>
    </div>
  </body>
</html>
```
4. Change directory to **Books*
1. Start the server using ``node server.js``
![](https://user-images.githubusercontent.com/18899718/117218101-2756da80-adc8-11eb-888c-9cf468e1a648.png)
1. In your AWS console, open security group and edit inbound rules to accept **port 3300**
We access our app using instance IP address and adding port 3300
![](https://user-images.githubusercontent.com/18899718/117218496-e01d1980-adc8-11eb-820a-ddebaf7ba4ca.png)