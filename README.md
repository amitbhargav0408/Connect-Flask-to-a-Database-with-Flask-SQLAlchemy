# Connect-Flask-to-a-Database-with-Flask-SQLAlchemy
By now you're surely familiar with the benefits of Python's SQLAlchemy library: the all-in-one solution for basically anything database related. Like most major Python libraries, SQLAlchemy has been ported into a version specifically compatible with Flask, aptly named Flask-SQLAlchemy.

Similar to the core SQLAlchemy package, Flask-SQLAlchemy provides an ORM for us to modify application data by easily creating defined models. Regardless of what your database of choice might be, Flask-SQLAlchemy will ensure that the models we create in Python will translate to the syntax of our chosen database. Given the ease-of-use and one-size-fits-all  nature of Flask-SQLAlchemy, it's no wonder that the library has been the de facto database library of choice for Flask since the very beginning (seriously, is there even another option?)

Configuring Flask-SQLAlchemy For Your Application
There are a few essential configuration variables we need to set upfront before interacting with our database. As is standard, we'll be using a class defined in config.py to handle our Flask config:

There are a few variables here which are specific to Flask-SQLAlchemy:

SQLALCHEMY_DATABASE_URI: the connection string we need to connect to our database. This follows the standard convention: [DB_TYPE]+[DB_CONNECTOR]://[USERNAME]:[PASSWORD]@[HOST]:[PORT]/[DB_NAME]
SQLALCHEMY_ECHO: When set to 'True', Flask-SQLAlchemy will log all database activity to Python's stderr for debugging purposes.
SQLALCHEMY_TRACK_MODIFICATIONS: Honestly, I just always set this to 'False,' otherwise an obnoxious warning appears every time you run your app reminding you that this option takes a lot of system resources.
Those are the big ones we should worry about. If you're into some next-level database shit, there are a few other pro-mode configuration variables which you can find here.

Initiating Flask-SQLAlchemy
As always, we're going to use the Flask Application Factory method for initiating our app. If you're unfamiliar with the term, you're going to find this tutorial to be confusing and pretty much useless.

The most basic __init__.py file for Flask applications using Flask-SQLAlchemy should look like this:

Note the presence of db and its location: this our database object being set as a global variable outside of create_app(). Inside of create_app(), on the other hand, contains the line db.init_app(app). Even though we've set our db object globally, this means nothing until we initialize it after creating our application. We accomplish this by calling init_app() within create_app(), and passing our app as the parameter. Within the actual 'application context' is where we'll call create_all(), which we'll cover in a bit.

If that last paragraph sounded like total gibberish to you, you are not alone. The Flask Application Factory is perhaps one of the most odd and poorly explained concepts in Python software development- my best advice is to not become frustrated, take the copy + paste code above, and blindly accept the spoon-fed nonsense enough times until it becomes second nature. That's what I did, and even as I worked through this tutorial, I still came across obnoxious quirks that caught me off-guard.

Take note of import we make inside of the application context called routes. This is one of two files we haven't written just yet: once we create them, our application file structure will look something like this:
over project structure
flask-sqlalchemy-tutorial
├── /application
│   ├── __init__.py
│   ├── routes.py
│   └── models.py
├── config.py
└── wsgi.py

Creating Database Models
Create a models.py file in our application directory. Here we'll import the db object that we created in __init__.py. Now we can create database models by defining classes in this file.

A common example would be to start with a User model. The first variable we create is __tablename__, which will correspond to the name of the SQL table new users will be saved. Each additional variable we create within this model class will correspond a column in the database:

Each "column" accepts the following attributes:

Data Type: Accepts one of the following: String(size), Text, DateTime, Float, Boolean, PickleType, or LargeBinary.
primary_key (optional): When set to True, this field will serve as the primary key in the corresponding database table.  
foreign_key (optional): Sets a foreign key relationship to a field in another table. The target column must have a unique constraint in order to build a relationship between two tables.
unique (optional): When set to True, the field will have a unique constraint added in the database.
nullable (optional):  Accepts a Boolean value which specifies whether or not this column can accept empty values.
With our first model created, you're already way closer to interacting with your database than you might think.

Creating Our First User
With our User model created, we now have what we need to create users in our database on the fly. Here we'll crack open routes.py and make a simple route to create a user. This route will accept two query string parameters: ?user= will accept a username, and &email= will accept an email address:

Check out how easy this is! All it takes to create a user is create an instance of the user class, add it to our session via db.session.add(new_user), and commit the changes with db.session.commit()! This is the appeal of powering applications with a database ORM like SQLAlchemy: instead of bothering with handling SQL statements, we can now manage our data as though they were objects. In the context of building an app, data models grant us a safe, easy way to modify data.

The new_user variable creates an instance of User locally in memory. We pass keyword arguments to set the values of each field in our class (username, email, created, and bio).

db.session.add(new_user) stages the new user we just created to be added to the database. As with any database interaction in Python, the change won't actually be committed until we explicitly give the OK. In Flask-SQLAlchemy, this is done via db.session.commit().

Querying Our New Data
We've created a user, but how can we make sure that duplicate records aren't created? Since the username and email fields have the unique constraint, our database technically won't allow this to happen. If we don't add the proper validation, attempting to insert multiple unique values into our table will cause the app to crash. Instead of crashing, let's build in a nice error message!
Before we create a new user, the existing_user variable queries our user table to see if a record already exists with the information provided. To query records which belong to a data model, we write a statement like this:
[MODEL] is our data model class name, which we follow this up with .query. Now, we can construct a query using a number of methods at our disposal:

filter([CRITERIA]): Return all records which match a given criteria.
order_by([MODEL].[COLUMN]): Sort returned records by a column.
get([VALUE]): Retrieve a record by its primary key.
.limit([INTEGER]): Set a maximum number of records to retrieve.
In our case, we use the .filter() method to retrieve existing users which already have our desired username or email taken (we use first() in our query instead of all(), because we're expecting a maximum of 1 record). If a record exists, we deliver an error to the user:
Loading Records Into a Template
For our final trick, let's see our Jinja plays along with database records. I've modified our route below to render a template called users.html. This template will create an HTML page which lists all existing users:The statement User.query.all() will return all instances of User in our database. We pass all of these records into our template with users=users. Here's our Jinja template:

