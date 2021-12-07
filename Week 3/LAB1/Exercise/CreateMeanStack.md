# Create MEAN Stack (https://docs.microsoft.com/en-us/learn/modules/build-a-web-app-with-mean-on-a-linux-vm/)

## Create a VM to host your web application

1) Create Ubuntu VM
```
az vm create \
  --resource-group learn-a7ad8b25-7915-4989-993a-a780da893e78 \
  --name MeanStack \
  --image Canonical:0001-com-ubuntu-server-focal:20_04-lts:latest \
  --admin-username azureuser \
  --generate-ssh-keys
```
Output:
```
{
  "fqdns": "",
  "id": "/subscriptions/741358db-4301-44e8-b1ce-a919346cceb5/resourceGroups/learn-a7ad8b25-7915-4989-993a-a780da893e78/providers/Microsoft.Compute/virtualMachines/MeanStack",
  "location": "westus",
  "macAddress": "00-22-48-08-78-F1",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "40.112.166.181",
  "resourceGroup": "learn-a7ad8b25-7915-4989-993a-a780da893e78",
  "zones": ""
}
```

2) Open port 80 on the VM to allow incoming HTTP traffic to the web application
```
az vm open-port \
  --port 80 \
  --resource-group learn-a7ad8b25-7915-4989-993a-a780da893e78 \
  --name MeanStack
```

3) Display your VM's public IP address
```
ipaddress=$(az vm show \
  --name MeanStack \
  --resource-group learn-a7ad8b25-7915-4989-993a-a780da893e78 \
  --show-details \
  --query [publicIps] \
  --output tsv)
```

4) SSH to VM
```
ssh azureuser@$ipaddress
```

## Install MongoDB

5) Make sure all current packages are up to date
```
sudo apt update && sudo apt upgrade -y
```

6) Install MongoDB
```
sudo apt-get install -y mongodb
```

7) Confirm service is running
```
sudo systemctl status mongodb
```

8) Verify MongoDB version
```
mongod --version
```

## Install node.js

9) Register the Node.js repository so the package manager can locate the packages (it can take about 10 mins)
```
curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -
```

10) Install Node.js package
```
sudo apt install nodejs
```

11) Run the following to verify installation
```
nodejs -v
```

12) Exit SSH session


## Create a basic web application

Here's how each component of the MEAN stack fits in.

- MongoDB stores information about books.
- Express routes each HTTP request to the appropriate handler.
- AngularJS connects the user interface with the program's business logic.
- Node.js hosts the server-side application.

13) Run these commands to create the folders and files for your web application.
```
cd ~
mkdir Books
touch Books/server.js
touch Books/package.json
mkdir Books/app
touch Books/app/model.js
touch Books/app/routes.js
mkdir Books/public
touch Books/public/script.js
touch Books/public/index.html
```

14) Open Code in the browser
```
code Books
```

### Create Data Model

15) Open app/model.js and add the following:
```
var mongoose = require('mongoose');
var dbHost = 'mongodb://localhost:27017/Books';
mongoose.connect(dbHost, { useNewUrlParser: true } );
mongoose.connection;
mongoose.set('debug', true);
var bookSchema = mongoose.Schema( {
    name: String,
    isbn: {type: String, index: true},
    author: String,
    pages: Number
});
var Book = mongoose.model('Book', bookSchema);
module.exports = Book;
```

### Create the Express routes that handle HTTP requests

16) Open app/routes.js and add the following code
```
var path = require('path');
var Book = require('./model');
var routes = function(app) {
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
    app.get('*', function(req, res) {
        res.sendFile(path.join(__dirname + '/public', 'index.html'));
    });
};
module.exports = routes;
```

### Create the client-side JavaScript application

17) open public/script.js and add this code
```
var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope, $http) {
    var getData = function() {
        return $http( {
            method: 'GET',
            url: '/book'
        }).then(function successCallback(response) {
            $scope.books = response.data;
        }, function errorCallback(response) {
            console.log('Error: ' + response);
        });
    };
    getData();
    $scope.del_book = function(book) {
        $http( {
            method: 'DELETE',
            url: '/book/:isbn',
            params: {'isbn': book.isbn}
        }).then(function successCallback(response) {
            console.log(response);
            return getData();
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
            return getData();
        }, function errorCallback(response) {
            console.log('Error: ' + response);
        });
    };
});
```

### Create User Interface

18) open public/index.html and add this code
```
<!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
<head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.7.2/angular.min.js"></script>
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
        <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
        <td>{{book.name}}</td>
        <td>{{book.isbn}}</td>
        <td>{{book.author}}</td>
        <td>{{book.pages}}</td>
        </tr>
    </table>
    </div>
</body>
</html>
```


### Create the Express server to host the application

19) open server.js and add this code
```
var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./app/routes')(app);
app.set('port', 80);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});
```

### Define package information and dependencies

20) open package.json and add this code.
```
{
  "name": "books",
  "description": "Sample web app that manages book information.",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/MicrosoftDocs/mslearn-build-a-web-app-with-mean-on-a-linux-vm"
  },
  "main": "server.js",
  "dependencies": {
    "express": "~4.16",
    "mongoose": "~5.3",
    "body-parser": "~1.18"
  }
}
```

21) Copy files to your VM
```
scp -r ~/Books azureuser@$ipaddress:~/Books
```

22) Set Public IP as variable 
```
ipaddress=$(az vm show \
  --name MeanStack \
  --resource-group learn-a7ad8b25-7915-4989-993a-a780da893e78 \
  --show-details \
  --query [publicIps] \
  --output tsv)
```

23) Show IP
```
echo $ipaddress
```

24) SSH to VM
```
ssh azureuser@$ipaddress
```

25) Move to Books directory:
```
cd ~/Books
```

26) Install dependent packages
```
sudo apt install npm
npm install
```

27) Start web app
```
sudo nodejs server.js
```

28) Open browser and enter public IP
  

