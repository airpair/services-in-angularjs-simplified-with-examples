## What is a service?

- It provides us method to keep data across the lifetime of the angular app 
- It provides us method to communicate data across the controllers in a consistent way
- This is a singleton object and it gets instantiated only once per application 
- It is used to organize  and share data and functions across the application 

Two main execution characteristics of angular services are that they are singleton and lazy instantiated. Angular only creates instance of a service when an application component depends on it. On the other hand each application component dependent on the service work with the single instance of the service created by the angular. 

![services](https://dhananjay25.files.wordpress.com/2015/02/image6.png)

Keep in mind that services are singleton object and gets instantiated once per angular app using the $injector and they gets created when angular app components need them. 
Let us start with creating a very simple service. This service will find square of a given number. It is a good idea to put all the service in a separate JavaScript file.  We have created service.js file and inside that creating a service or registering a service using the service method as shown below, 

```javascript
var CalculatorService = angular.module('CalculatorService', [])
.service('Calculator', function () {
    this.square = function (a) { return a*a};
    
});
```
An AngularJS service can be created or registered or created in four different ways, 

- Using the service() method
- Using the factory() method 
- Using the provider() method
- Using the value() method
- Using the constant() method 

Further in the post we will talk about options to create the service.  Above we have created the calculator service using the service() method. 
To use the service in the controller, we are3 passing the service module CalculatorService as dependency in the application module. Next in the controller we are passing name of the service Calculator to be used. 

```javascript
var myApp = angular.module('myApp', ['CalculatorService']);
myApp.controller('CalculatorController', function ($scope, Calculator) {

    $scope.findSquare = function () {
        $scope.answer = Calculator.square($scope.number);
    }
});

```

On the view we are using the controller to do the data binding as shown below, 

```html
<div class="container">
        <div>
            <div ng-controller="CalculatorController">
                Enter a number:
                <input type="number" ng-model="number">
                <button  ng-click="findSquare()">Square</button>
                <div>{{answer}}</div>
            </div>
        </div>
    </div>

```

We can create a service using the factory as shown below. We are creating the service to reverse the string. 

```javascript


CalculatorService.factory('StringManipulation', function () {

   var r=  function reverse(s) {
        var o = '';
        for (var i = s.length - 1; i >= 0; i--)
            o += s[i];
        return o;
    }
    
   return{
       reverseString: function reverseString(name)
       {
           return r(name);
       }
   }

});
```

In the controller the StringManipulationService can be used as shown below:

```javascript
myApp.controller('CalculatorController', function ($scope, Calculator) {

    $scope.findSquare = function () {
        $scope.answer = Calculator.square($scope.number);
    }
});


```

On the view we are using the controller to do the data binding as shown below, 

```html
  <div ng-controller="StringController">
                Enter Name:
                <input ng-model="name">
                <button  ng-click="findReverse()">Reverse Name</button>
                <div>{{reversename}}</div>
            </div>

```

Let us understand difference between creating a service using the service() method and the factory() method. 

- Using the service() method uses the function constructor and it returns the object or instance of the function to work with 
- Using the factory() method uses the returned value of the function. It returns the value of the function returned after the execution 

If we want to register a service using the function constructor, in that scenario we will use service()  method to register the service. If we use factory() method to register the service it returns the value after execution of the service function. It can return any value like primitive value, function or object. So service() method returns the function object whereas factory() method can return any kind of value. 

In further post we will talk about other ways of creating service. Now let us work on a full working example of using service in an angular app. We are going to create an application which will perform the followings, 

- List students from the database
- Add a student to the database 


This post will not cover how to create a JSON based WCF REST Service. With the assumption that REST based service is already in place to retrieve the students and add a student, we will write the angular application. 

##The Service 

~~~javascript

var StudentService = angular.module('StudentService', [])
StudentService.factory('StudentDataOp', ['$http', function ($http) {

    var urlBase = 'http://localhost:2307/Service1.svc';
    var StudentDataOp = {};

    StudentDataOp.getStudents = function () {
        return $http.get(urlBase+'/GetStudents');
    };

    StudentDataOp.addStudent = function (stud) {
        return $http.post(urlBase + '/AddStudent', stud);
    };
    return StudentDataOp;

}]);


~~~

We are creating the service using the factory(). There are two methods in the service. getStudents fetch all the students  using the $http.get whereas addStudent add a student using the $http.post. In the service we are using other inbuilt angular service $http to make the call to the service. To use the $http service, we have to pass this as dependency to the service factory() method. 
Once service is created, let us create the controller which will use the service to perform the operations. 

###The Controller 

~~~javascript 
var myApp = angular.module('myApp', ['StudentService']);

myApp.controller('StudentController', function ($scope, StudentDataOp) {
    $scope.status;
    $scope.students;
    getStudents();

    function getStudents() {
        StudentDataOp.getStudents()
            .success(function (studs) {
                $scope.students = studs;
            })
            .error(function (error) {
                $scope.status = 'Unable to load customer data: ' + error.message;
            });
    }

    $scope.addStudent = function () {
        
        var stud = {
            ID: 145,
            FirstName: $scope.fname,
            LastName: $scope.lname
        };
        StudentDataOp.addStudent(stud)
            .success(function () {
                $scope.status = 'Inserted Student! Refreshing Student list.';
                $scope.students.push(stud);
            }).
            error(function (error) {
                $scope.status = 'Unable to insert Student: ' + error.message;
            });
    };
});


~~~

In the controller we are adding getStudents and addStudents functions to scope. As it is clear from the name that these functions are used to fetch students and add student respectively. As a dependency StudentSevice module is passed in the module and in the controller we are passing the StudentDataOp service as the dependency. 
Other important thing to notice is that, we are creating student object to be added using the $scope object properties like fname and lname. These properties are set to the $scope on the view. 

###The View

View is very simple. StudentController is attached to the view. There are two section in the view. In first section we are taking user input to create the student. In the second section, in a table all the students are listed. 

~~~html

<div ng-controller="StudentController">
            <form class="well">               
                <input type="text" name="fname" ng-model="fname" placeholder="first name" /> <br/>       
                <input type="text" name="lname" ng-model="lname" placeholder="last name" />             
                <br /><br/>               
                <input type="button" value="add student" ng-click="addStudent()" />
            </form>
            <br/>
            <table class="table">
                <tr ng-repeat="s in students">
                    <td>{{ s.FirstName }}</td>
                    <td>{{ s.LastName }}</td>
                </tr>
            </table>
        </div>

~~~

In the input form we are setting fname and lname properties on the $scope object and then calling addStudent() function of the controller to add a student to the database. 
In the table using the ng-repeat directive all the students are listed. We have put two buttons for edit and delete. These buttons are not performing any task as of yet. In further posts we will add edit, delete and search functionality.

This is how you can create and use angularJS service in application. I hope you find this post useful. Thanks for reading. Happy coding.  




