# **MEAN STACK DEPLOYMENT TO UBUNTU IN AWS**

The main aim of this project is to build a book register application that has the front end using angular framework and also have a back end express that stores information on a database which is MongoDB that is running on a Node.js Javascript environment on the browser. All these will be done on AWS using Ubuntu operating system

**MEAN** stack is a combination of the following components:
- **M**ongoDB - open-source, NoSQL (non-relational) database management system. It is designed to store and manage large volumes of unstructured data.
- **E**xpress - a popular and minimalist web application framework for Node.js, a JavaScript runtime environment. It provides a robust set of features and tools for building web applications and APIs on the server-side.
- **A**ngular - It is used for building dynamic and robust web applications with a focus on Single Page Applications (SPAs).
- **N**ode.js - an open-source JavaScript runtime environment that allows developers to run JavaScript code on the server-side.

## **Spinning up EC2 instance using Ubuntu**

[EC2 Instance]![ubuntu](https://github.com/Lummysloane/Devops-Project/assets/131771280/b68af1ed-e470-4160-bf04-a03940c1c5ce)

## **Connected to the server via Ubuntu**

![serverconnect](https://github.com/Lummysloane/Devops-Project/assets/131771280/f4b0f718-bb6f-4ba3-a60c-88dcc5f0ec42)

# IMPLEMENTING A SIMPLE BOOK REGISTER WEB FORM USING MENA STACK

*STEP 1* - Start by ## **Installing NodeJs**

Node.js is an open-source JavaScript runtime environment that allows developers to run JavaScript code on the server-side. Node.js provides an event-driven, non-blocking I/O model, making it highly efficient and scalable for building server-side applications. It utilizes an event loop to handle multiple concurrent requests without blocking the execution of other operations, resulting in excellent performance and responsiveness.

But first we have to update Ubuntu `sudo apt update` and upgrade Ubuntu `sudo apt upgrade`

![sudoupdateupgrade](https://github.com/Lummysloane/Devops-Project/assets/131771280/af462b99-c989-49fd-9973-ecf5dbc960a3)

## Install nodejs

`sudo apt install -y nodejs`

![installnodejs](https://github.com/Lummysloane/Devops-Project/assets/131771280/8f12a170-d493-40b7-8e63-ae018e05ead1)

*STEP 2* - **Install MongoDB**

**MongoDB**is designed to handle large volumes of unstructured and semi-structured data, making it particularly suitable for modern applications with flexible data requirements.

To install MondoDB, we have to add MongoDB key to our key server

`sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6`

then, add a repository for MongoDB

`echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list`

**Install MongoDB**

`sudo apt install -y mongodb`

**Start Server**

`sudo service mongodb start`

There is a service that runs on every background of every application to ensure that this program is running effectively. it monitors and also keeps logs of all the services that the application is rendering. it is always important to start the service

To verify that the service is up and running 

`sudo systemctl status mongodb`

![mongodbactive](https://github.com/Lummysloane/Devops-Project/assets/131771280/1c1eebb8-4257-4c69-8852-fec7e2684153)

**Installing npm**

NPM (Node Package Manager) is the default package manager for Node.js, providing a vast ecosystem of reusable libraries and tools that can be easily integrated into Node.js projects. It allows developers to install, manage, and share packages and dependencies for their Node.js applications.

`sudo apt install -y npm`

**Installing Body-parser**

Body-parser is a middleware module for Node.js web applications that simplifies the handling of HTTP request bodies. NPM helps to install Body-parser

`sudo npm install body-parser`

![bodyparser](https://github.com/Lummysloane/Devops-Project/assets/131771280/9b51d451-a3fd-49fd-a212-0f629d0ee00a)

We are going to create a folder where we are going to build our application. That directory will be called *books*. After making the directory *books*, we are going to cd into it.

`mkdir Books && cd Books`

![mkdirbook](https://github.com/Lummysloane/Devops-Project/assets/131771280/8542bc24-7ddf-4bd2-9e94-fb54d41a828f)

In the Books directory, Initialize npm project

`npm init`

Add a file to it named *server.js*

`vi server.js` (**vi is text editor on linux environment**)

Copy and paste the web server code below into the *server.js* file.

**var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./apps/routes')(app);
app.set('port', 3300);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});**

![serverjs](https://github.com/Lummysloane/Devops-Project/assets/131771280/48f323ad-61fa-482e-90a9-bc9d0c1d67a6)


*Step 3* - **Install Express and set up routes to the server**

Express is a fast and minimalist web application framework for Node.js, designed to simplify the process of building robust and scalable web applications and APIs. It provides a set of features and tools that make it easier to handle HTTP requests, define routes, and manage middleware.

We also will use Mongoose package which provides a straight-forward, schema-based solution to model your application data. We will use Mongoose to establish a schema for the database to store data of our book register.

**To install express mongoose**

`sudo npm install express mongoose`

![installmongoose](https://github.com/Lummysloane/Devops-Project/assets/131771280/4321b7f3-3134-4084-a8e6-0950a0835054)

In ‘Books’ folder, create a folder named apps

`mkdir apps && cd apps`

Create a file named routes.js

`vi routes.js`

Copy and paste the code below into routes.js

**const Book = require('./models/book');
module.exports = function(app){
  app.get('/book', function(req, res){
    Book.find({}).then(result => {
      res.json(result);
    }).catch(err => {
      console.error(err);
      res.status(500).send('An error occurred while retrieving books');
    });
  });
app.post('/book', function(req, res){
    const book = new Book({
      name: req.body.name,
      isbn: req.body.isbn,
      author: req.body.author,
      pages: req.body.pages
    });
    book.save().then(result => {
      res.json({
        message: "Successfully added book",
        book: result
      });
    }).catch(err => {
      console.error(err);
      res.status(500).send('An error occurred while saving the book');
    });
  });
app.delete("/book/:isbn", function(req, res){
    Book.findOneAndRemove(req.query).then(result => {
      res.json({
        message: "Successfully deleted the book",
        book: result
      });
    }).catch(err => {
      console.error(err);
      res.status(500).send('An error occurred while deleting the book');
    });
  });
const path = require('path');
  app.get('*', function(req, res){
    res.sendFile(path.join(__dirname, 'public', 'index.html'));
  });
};**

In the ‘apps’ folder, create a folder named models

`mkdir models && cd models`

![bookjs](https://github.com/Lummysloane/Devops-Project/assets/131771280/e913d61e-4a6e-46b8-9493-a1ccb243f634)


Add a file named script.js

`vi script.js`

![scriptjs](https://github.com/Lummysloane/Devops-Project/assets/131771280/a26d3877-6ffe-45a4-868e-ca973dd544d0)

Copy and paste the Code below (controller configuration defined) into the script.js file.

**var app = angular.module('myApp', []);
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
});**

In public folder, create a file named index.html

`vi index.html`

![indexjs](https://github.com/Lummysloane/Devops-Project/assets/131771280/4328daaf-8fd7-419f-8f40-a7e7ed431e49)

Copy and paste the code below into index.html file.

<!doctype html>
html ng-app="myApp" ng-controller="myCtrl">
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

Change the directory back up to Books

`cd ..`

Start the server by running the command below

`node server.js`

![nodestartup](https://github.com/Lummysloane/Devops-Project/assets/131771280/5e06b9b0-b172-456c-8134-5c881801f059)

The server is now up and running but we need to open TCP port 3300 in our AWS Web Console for our EC2 Instance.

![3300](https://github.com/Lummysloane/Devops-Project/assets/131771280/8e6f11ba-f03d-499d-9019-664148773416)

Now you can access our Book Register web application from the Internet with a browser using Public IP address or Public DNS name

![website](https://github.com/Lummysloane/Devops-Project/assets/131771280/6d40fe1d-a151-4171-be7f-d033bb30e37d)















































