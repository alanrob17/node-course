## API Authentication and Security

### Securely Storing Passwords: Part I

We are going to learn how to securely store user passwords by hashing and salting the password before storing it in the database.

#### Hashing Passwords

Storing plain text passwords is a bad idea. Most folks reuse passwords for multiple accounts online. That means if your database gets compromised, the hacker can reuse those credentials on other sites such as credit cards or bank accounts. We don't want to leave our users open to further attacks.

The solution is to hash passwords using a secure one-way hashing algorithm. Users passwords will stay hidden and secure, even if the database is compromised. 

#### Hashing Passwords with Bcrypt

First up, install the library.

```
npm i bcryptjs
```

The **hash** method can be used to hash the plain text password. The example below hashes the password “Red12345!”.

```
    const password = 'Red12345!';

    // The hashed password is what would be stored in the database
    const hashedPassword = await bcrypt.hash(password, 8);
```

The **compare** method is used to compare a plain text password against a previously hashed password. This would be useful when logging in. The user logging in provides the plain text password for their account. The application fetches the hashed password from the database for that user. compare is then called to confirm it's a match.

```
    const isMatch = await bcrypt.compare('red12345!', hashedPassword);
    console.log(isMatch);
```

My example code.

```
    const bcrypt = require('bcryptjs');

    const myFunction = async () => {
        const password = "Red12345%";

        const hashPassword = await bcrypt.hash(password, 8);

        console.log(password);
        console.log(hashPassword);
    }

    myFunction();
```
 Results:

> Red12345%  
> $2a$08$9pHmqFNVD37bB1H2gDKzPOePz9W6tGuOtnLVrpLXjY0j/.jAtsIna  

Hashing algorithms are by design not reversible. This means that there is no decryption method.

All we can do is compare a hashed password in the database against the password entered in the input field.

```
    const bcrypt = require('bcryptjs');

    const myFunction = async () => {
        const password = "Red12345%";

        const hashPassword = await bcrypt.hash(password, 8);

        console.log(password);
        console.log(hashPassword);

        const isMatch = await bcrypt.compare('Red12345%', hashPassword);

        if (isMatch) {
            console.log('Passwords matched!');
        } else {
            console.log('Passwords didn\'t match!!!!');
        }
    }

    myFunction();
```

> Passwords matched!

### Securely Storing Passwords: Part II

We are now going to learn how to use Mongoose middleware. Middleware will allow you to automatically hash a user's password before the user is saved to the database.

#### Mongoose Middleware

Middleware allows you to register some code to run before or after a lifecycle event for your model. As an example, you could use middleware to register some code to run just after a user is deleted. You could also use middleware to register some code to run just before the user is saved. This can be used to hash passwords just before saving users to
the database.

The example below calls **pre** with the ``'save'`` lifecycle event. This registers a function to run just before users are saved. The function itself checks if the password has been altered. If the password has been altered, the plain text password is overwritten with a hashed version.

```
    userSchema.pre('save', async function (next) {
        const user = this;

        if (user.isModified('password')) {
            user.password = await bcrypt.hash(user.password, 8);
        }

        next();
    });
```

In our example we are going to change the User code so that we can store a hashed password into our database.

We have to make changes to our user model and our user router.

##### /src/models/user.js

```
    const mongoose = require('mongoose');
    const validator = require('validator');
    const bcrypt = require('bcryptjs');

    const userSchema = new mongoose.Schema({
        name: {
            type: String,
            required: true,
            trim: true
        },
        age: {
            type: Number,
            default: 0,
            validate(value) {
                if (value < 0) {
                    throw new Error('Age must be a positive number!');
                }
            }
        },
        email: {
            type: String,
            required: true,
            trim: true,
            lowercase: true,
            validate(value) {
                if (!validator.isEmail(value)) {
                    throw new Error('Failed to provide a valid email address!');
                }
            }
        },
        password: {
            type: String,
            required: true,
            trim: true,
            validate(value) {
                if (!validator.isLength(value, {min: 6, max: undefined})) {
                    throw new Error('password must be more than 6 characters!');
                } else if (validator.contains(value.toLowerCase(), 'password')) {
                    throw new Error('Invalid password!');
                }
            }
        }
    });

    userSchema.pre('save', async function (next) {
        const user = this;

        if (user.isModified('password')) {
            user.password = await bcrypt.hash(user.password, 8);
        }

        next();
    });

    const User = mongoose.model('User', userSchema);

    module.exports = User;
```

By adding the ``userSchema.pre()`` we are adding in Middleware. Every time a user record is added or modified the ``userSchema.pre()`` middleware method will run and allow us to encrypt the user's password.

##### /src/routers/user.js - update route

```
    router.patch('/users/:id', async (req, res) => {
        const updates = Object.keys(req.body); 
        const allowedUpdates = ['name', 'email', 'password', 'age'];

        const isValidOperation = updates.every((update) => allowedUpdates.includes(update));

        if (!isValidOperation) {
            return res.status(400).send({error: 'Invalid updates.'});
        }

        try {
            const user = await User.findById(req.params.id);


            updates.forEach((update) => user[update] = req.body[update]);
            await user.save();

            if (!user) {
                return res.status(404).send();
            }

            res.send(user);
        } catch (e) {
            res.status(400).send(e);
        }
    });

    router.delete('/users/:id', async (req, res) => {
        try {
            const user = await User.findByIdAndDelete(req.params.id)

            if (!user) {
                return res.status(404).send();
            }

            res.send(user);
        } catch (e) {
            res.status(500).send();
        }
    });
```

We are using middleware in the **Update** route and to get this to work we had to make changes from ``User.findByIdAndUpdate()`` to ``User.findById()`` because the ``User.findByIdAndUpdate()`` doesn't work with the Mongoose middleware.

In Postman we add a new user.

![Postman Create new user with hashed password](assets/images/postman-add-new-user.jpg "Postman Create new user with hashed password")

In the database.

> /* 6 */  
> {  
>     "_id" : ObjectId("5ef03a526a97684a3490bf66"),  
>     "age" : 0,  
>     "name" : "Jen Robson",  
>     "email" : "jen@jen.com.au",  
>     "password" : "$2a$08$z1ugeu271rQDnQnbidsy8.gYzCxRTDCfXRwziCdO2ps0RVJ.DQXjm",  
>     "__v" : 0  
> }  

We can also update a document in Postman.

![Postman Update user with hashed password](assets/images/postman-add-new-user.jpg "Postman Update user with hashed password")

In the database.

> /* 5 */  
> {  
>     "_id" : ObjectId("5ef0359180589330a052f47b"),  
>     "age" : 34,  
>     "name" : "Ethan James Robson",  
>     "email" : "ethan@ethan.com.au",  
>     "password" : "$2a$08$n/oVuMWLCnGtDNpoM0L3Gepc.zDHEVhMIm.fcf7UGiuuWSh/pMR6q",  
>     "__v" : 0  
> }  

##### Challenge

Change how the tasks are updated.

1. Find the task.
2. Alter the task properties
3. Save the task.
4. Test your work by updating a task in Postman.

![Postman Update task to allow for middleware](assets/images/postman-update-task-middleware.jpg "Postman Update task to allow for middleware")

### Logging in Users

We need the ability for users to log into their existing accounts.

#### Logging in Users

Logging in a user is a two-step process. The user provides their email and password, and the first thing to do is fetch the user by their email. From there, bcrypt is used to verify the password provided matches the hashed password stored in the database. If either step fails, the users won't be able to log in. If both steps succeed, then you know the user is
who they say they are.

We need to create a route that allows a user to enter their credentials and then validate that they are indeed that user.

We need to add the route in ``routers/user.js``.

```
    router.post('/users/login', async (req, res) => {
        try {
            const user = await User.findByCredentials(req.body.email, req.body.password);

            res.send(user);
        } catch (e) {
            res.status(400).send();
        }
    });
```

We now need to create our own function, ``findByCredentials()`` that is part of the **User** model. We can do this in the **User** model with ``userSchema.statics``.

```
    userSchema.statics.findByCredentials = async (email, password) => {
        const user = await User.findOne(email);

        if (!user) {
            throw new Error('Unable to login!');
        }

        const isMatch = bcrypt.compare(password, user.password);

        if (!isMatch) {
            throw new Error('Unable to login!');
        }

        return user;
    }
```

**Note:** we are providing a very generic error message that doesn't give away too much information to a malicious user. We don't want them to know that the email was valid and that the password failed. This is why both errors have the same message.

We also need to allow for unique email addresses. Only one email address should appear in our database. We can do this by changing the **User** model.

```
    email: {
        type: String,
        unique: true,
        required: true,
        trim: true,
        lowercase: true,
        validate(value) {
            if (!validator.isEmail(value)) {
                throw new Error('Failed to provide a valid email address!');
            }
        }
    },
```

We add the **unique: true** property.

**Note** there is a catch to this. We have to delete all **User** documents from our database to enable this to happen.

We are actually going to drop the whole database.

**Note:** we have to stop and start our web server as well!

Once this has been completed we can go over to Postman and create a new **User** and a new **Task**.

Go back to the database and refresh ``Ctrl-R`` and you will see that the database and the two collections have been re-created.

Once again, in Postman try and add the same user record you added previously to make sure that a user can't create a document with the same email address.

We get this message.

![Postman duplicate email error](assets/images/duplicate-email.jpg "Postman duplicate email error")

Once we are happy that a user can't create a duplicate email account we can create a new **login** request in Postman.

I have created a user with the credentials.

> alan@alan.com.au  
> tulsa@123  

In Postman I run the user login with the credentials above and I get.

![Postman successful login](assets/images/postman-successful-login.jpg "Postman successful login")

### JSON Web Tokens

 We are now going to explore JWT's (JSON Web Tokens). JWT's provide a nice system for issuing and validating authentication tokens. The authentication token will ensure that the client doesn't need to log in every time they want to perform an operation on the server.

#### JSON Web Tokens

First up, install the library.

```
    npm i jsonwebtoken
```

The **sign** method can be used to generate a new token. **sign** accepts three arguments: The first is the data to embed in the token: This needs to include a unique identifier for the user. The second is a secret phrase: This is used to issue and validate tokens, ensuring that the token data hasn't been tampered with. The third is a set of options: The example below uses **expiresIn** to create a token that's valid for seven days.

```
    const jwt = require('jsonwebtoken');

    const token = jwt.sign({ _id: '1234' }, 'thisismynodecourse', { expiresIn: '7 days' });
```
Tokens can be issued to users when they sign up or log in to the application. These can then be stored on the data and used to authenticate the user when they perform other options.

Once you have created a token you will notice that there are two periods in the string. These denote three different parts. The first part is known as the header. The second part is the payload or body and it contains the data we provided. In our case it would be the id we provided. The last part is the signature and it is used to verify the token. The point of the token isn't to hide the data and it is publicly viewable. The whole point is to create data (our id) that is verifiable via the signature (the second value in the sign in process). So if somebody comes along and tries to change the body they are not going to be able to do this because they won't know what secret (signature) was used.

We can copy the middle value and put it into the following site.

[BASE64  website](https://www.base64decode.org "BASE64  website")

It will decode the following data.

> {"_id":"1234","iat":1592876517}

As you can see the ``_id`` is publicly available as well as the timestamp it was created.

We have seen how to create a token and now we will see how to verify it.

The server can verify the token using verify. This requires two arguments: The first is the token to validate. The second is the secret phrase that the token was created with. If valid, the embedded data will be returned. This would allow the server to figure out which user is performing the operation.

```
    // data._id contains the user id of the user that owns this token
    const data = jwt.verify(token, 'thisismynodecourse');
```

{ _id: '1234', iat: 1592878315 }

``verify()`` is going to return the payload for the token if everything went well or throw an error if there were problems.

If we mess with the signature and add an extra character to it we will get our error.

> (node:10572) UnhandledPromiseRejectionWarning: JsonWebTokenError: invalid signature

We can also add the expiration data to the ``sign()`` method.

```
    const jwt = require('jsonwebtoken');

    const myFunction = async () => {
        const token = jwt.sign({ _id: '1234'}, 'thisismynodecourse');

        console.log(token);

        const data = jwt.verify(token, 'thisismynodecourse', { expiresIn: '7 days' });

        console.log(data);
    }

    myFunction();
```

> eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiIxMjM0IiwiaWF0IjoxNTkyODgwMDIzfQ.eeb2SR1IGZtXnox-tiRPgKEmtQG0IZoVvx52D1o0Kf4  
> { _id: '1234', iat: 1592880023 }  

### Generating Authentication Tokens

We are now going to integrate JWT's into the application. This will allow the app to issue an authentication token when a user signs up or logs in.

#### Generating and Storing Auth Tokens

Authentication tokens for a user can be stored in the database. This provides a way for users to log out. All generated authentication tokens will be stored as part of the user
profile. If a user logs out, that token will be removed from the user profile. A token would only be considered valid if it's a valid JWT and it's still stored as part of the user profile. A
user could be logged out of all sessions by simply deleting all the tokens stored in their user profile.

The snippet below adds a tokens array onto the user model. This will be used to store all valid authentication tokens for a user.

```
    const userSchema = new mongoose.Schema({
        tokens: [{
            token: {
                type: String,
                required: true
            }
        }]
    });
```

The instance method below is responsible for generating new authentication tokens. The token is created, stored in the database, and finally returned from the function.

```
    userSchema.methods.generateAuthToken = async function () {
        const user = this;

        const token = jwt.sign({ _id: user._id.toString() }, 'thisismynodecourse');

        return token;
    }
```

``generateAuthToken()`` can then be called to generate a fresh authentication token when users sign up or login.

```
    router.post('/users/login', async (req, res) => {
        try {
            const user = await User.findByCredentials(req.body.email, req.body.password);
            const token = await user.generateAuthToken();

            res.send({ user, token });
        } catch (e) {
            res.status(400).send();
        }
    });
```

When we run Postman we get the user object and the token that was created.

![Postman login showing user and token](assets/images/postman-show-user-token.jpg "Postman login showing user and token")

The token as it is at the moment has been passed to the client. We are not saving it on the server as yet and users can't log out so we can't expire the token. We can fix that by tracking the tokens we generate for users. This will allow a user to login from multiple devices, e.g a computer or a phone and then they would be allowed to logout on one while still being logged in on the other. So what we are going to do is store the token as part of the user document.

In the userSchema we add the **tokens** array.

```
    tokens: [{
        token: {
            type: String,
            required: true
        }
    }]
```

This will store multiple tokens for a user. In the ``generateAuthToken()`` method we need to concatenate the tokens into the database.

```
    userSchema.methods.generateAuthToken = async function () {
        const user = this;

        const token = jwt.sign({ _id: user._id.toString() }, 'thisismynodecourse');

        user.tokens = user.tokens.concat({ token });
        await user.save();

        return token;
    }
```

It is time to delete the database again and restart the application.

Now, back in Postman run the login again.

![Postman login saving token to the database](assets/images/postman-save-token-in-database.jpg "Postman login saving token to the database")

When we go back to the database we can see the users document now contains a single token.

```
    /* 2 */
    {
        "_id" : ObjectId("5ef063e174606f514cd361fd"),
        "age" : 0,
        "name" : "Alan Robson",
        "email" : "alan@alan.com.au",
        "password" : "$2a$08$vZT2nuzrm7HzsVJxdqjSBOMCqop6b5fPHdlDPxWc57UOxlUM31TiO",
        "__v" : 1,
        "tokens" : [
            {
                "_id" : ObjectId("5ef1b864cdc88d0ea0ec9020"),
                "token" : "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.>   eyJfaWQiOiI1ZWYwNjNlMTc0NjA2ZjUxNGNkMzYxZmQiLCJpYXQiOjE1OTI4OTk2ODR9.>    mbSdFgQjmLFw519wSdRMverv0GsNoKPZcjwQDF697UI"
            }
        ]
    }
```

##### Challenge

Have a new user signup generate authentication tokens.

1. Generate a new token for the saved user.
2. Send back both the token and the user.
3. Create a new user from Postman and confirm the token is there.

##### Modify Create user endpoint

```
    router.post('/users', async (req, res) => {
        const user = new User(req.body);

        try {
            await user.save();
            const token = await user.generateAuthToken();

            res.status(201).send({ user, token });  
        } catch (e) {
            res.status(400).send(e);  
        }
    });
```

![Postman Create user with token](assets/images/postman-new-user-with-token.jpg "Postman Create user with token")

In the database.

```
    /* 2 */
    {
        "_id" : ObjectId("5ef1bd5c815a5913701326a9"),
        "age" : 14,
        "name" : "James Robson",
        "email" : "james@james.com.au",
        "password" : "$2a$08$4tvbvsOggJMr2ECFHvdE0eQ6ogdn9Fa9mYY0b1kWCgF3lqOxM8H/K",
        "tokens" : [
            {
                "_id" : ObjectId("5ef1bd5c815a5913701326aa"),
                "token" : "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.    eyJfaWQiOiI1ZWYxYmQ1YzgxNWE1OTEzNzAxMzI2YTkiLCJpYXQiOjE1OTI5MDA5NTZ9.   a7NRUyVOGEM36R6BaJOaxNlCdGRzBgTOtZXEG2kJpKg"
            }
        ],
        "__v" : 0
    }
```

### Express Middleware

We are now going to explore Express middleware. When working with middleware, you'll have more control over how your server processes requests. This will be used to check that a user is authenticated before performing specific operations.

[Express Middleware web page](http://expressjs.com/th/guide/using-middleware.html "Express Middleware web page").

#### Exploring Express Middleware

Express middleware is nothing more than a function that runs as Express handles a given request. You can customize the function to do whatever you want it to do, and you can have it run whenever you want it to.

#### How middleware works

> Without middleware --> new request --> run route handler  
>   
> With middleware --> new request --> do something --> run route handler  

The example below uses middleware to print information about incoming requests. Middleware functions should accept three parameters: **req, res**, and **next**. The only new parameter is **next**. next is called to signal to Express that the middleware function is done.

```
    const loggerMiddleware = (req, res, next) => {
        console.log('New request to: ' + req.method + ' ' + req.path);
        next();
    }
```

Register your function as middleware for the application.

```
    app.use(loggerMiddleware);
```

#### A middleware function for Task app

This function is going to be used in **index.js**. It **must** be placed before all other ``app.use()`` statements.

```
    app.use((req, resp, next) => {
        console.log(req.method, req.path);
    });
```

This will log each request method and path. It will print the request method and the path but the way it is set up it will never finish. We need to add a ``next()`` method to complete the process.

```
    app.use((req, resp, next) => {
        console.log(req.method, req.path);
        next()
    });
```

The ``next()`` call will let the application know that we have finished our middleware process and ``next()`` allows the application to move on to run the route handler.

Now the process will complete and in our terminal we can see our request method and request path.

> GET /users

Once the middleware method has completed the route handler will process its process and in Postman you will see that all users have been printed out.

There are times when you wouldn't want your middleware to finish and in these cases you wouldn't want to proceed to ``next()``. This would be the case if your user couldn't be logged in for some reason.

Now for another example we will disable all GET requests and enable POST, PATCH or DELETE requests.

```
    app.use((req, res, next) => {
        if (req.method === 'GET') {
            res.send('GET requests are disabled!');
        } else {
            next();
        }
    });
```

We have to add in the ``res.send()`` call otherwise the GET call would never finsh. This results in the following Postman message.

![Postman GET requests disabled](assets/images/postman-get-requests-disabled.jpg "Postman GET requests disabled")

If we call a POST request to add a new user it will succeed.

This allows us to limit what a user has access to and we will use this in our application to enable authentication.

##### Challenge

Setup middleware for when your site is in maintenance mode.

1. Register a new middleware function.
2. Send back a maintenance message with a 503 code.
3. Try your requests from the server and confirm status/message is shown.

```
    const siteDown = true;

    app.use((req, res, next) => {
        if (siteDown) {
            res.status(503).send('Site is down for maintenance.');
        } else {
            next();
        }
    });
```

This results in the following Postman message.

![Postman Site is down message](assets/images/postman-site-down.jpg "Postman Site is down message")

These middleware requests will run for every route handler and we can also create middleware for specific requests only.

### Accepting Authentication Tokens

Now we can use Express middleware to put specific routes behind authentication. That will require the client to be authenticated before the operation can be performed. 

#### Accepting and Validating Tokens

The goal of the authentication middleware is to validate the authentication token and then fetch the profile for that user.**auth** below shows how you can get this done. Notice that the user profile is added onto **req.user**. This allows route handler functions to access the user profile without needing to fetch it again.

To do this we are going to create a new folder named middleware in src. We will create a new file for each piece of middleware for each task we create. In our case ``auth.js`` will be used for authentication.

We are going to add **auth** to the GET users route handler, i.e. you have to be authenticated to use this route handler.

##### src/middleware/auth.js

```
    const auth = async ((req, res, next) => {
        console.log('Authentication middleware is running.');
        next();
    });

    module.exports = auth;
```

At present we are just going to send a message that our middleware is working.

In ``src/routers/user.js`` we can add **auth** to our GET users route handler.

```
router.get('/users', auth, async (req, res) => {
    try {
        const users = await User.find({});
        res.send(users);
    } catch (e) {
        res.status(500).send();
    }
});
```

When we request **GET users** in Postman this will show all users and will print the following message in the terminal.

> Authentication middleware is running.

Now we are going to add the authentication.

```
    const User = require('../models/user');
    const jwt = require('jsonwebtoken');

    const auth = async (req, res, next) => {
       try {
        const token = req.header('Authorization').replace('Bearer ', '');
        console.log(token);
       } catch (e) {
           res.status(401).send({ error: 'Please authenticate.'});
       }
    };

    module.exports = auth;
```

To test this we have to have a JWT token for a logged in user. You can login as a user and at the bottom of the ``response.body`` grab the token.

We need to add this into the header of the **Read users** request.

![Postman Setting up authorisation](assets/images/postman-authorisation.jpg "Postman Setting up authorisation");

**Note: ** I used the English spelling for Authorisation and this has caused problems. I have now gone back and spelled it as Authorization. Even though this was only a variable it caused problems and once I gave it American spelling the problem was fixed.

If you look at your terminal you should see the token showing.

Now, once we have the token we decode it and this will give us the user id so we can then use this and the token to get the correct user details and verify that the user is logged in so we can show all of the user data to that specific user.

```
    const User = require('../models/user');
    const jwt = require('jsonwebtoken');

    const auth = async (req, res, next) => {
       try {
        const token = req.header('Authorization').replace('Bearer ', '');
        const decoded = jwt.verify(token, 'thisismynodecourse');
        const user = await User.findOne({ _id: decoded._id, 'tokens.token': token });

        req.user = user;

        next();
        if (!user) {
            throw new Error();
        }
       } catch (e) {
           res.status(401).send({ error: 'Please authenticate.'});
       }
    };

    module.exports = auth;
```

![Postman Successful validation](assets/images/postman-success-validation.jpg "Postman Successful validation");

If the user can see all of the user data it means that the user authenticated correctly.

Now we are going to mess with the authentication string and add a new character at the start of the string.

![Postman Unsuccessful validation](assets/images/Postman-unsuccessful-authentication.jpg "Postman Unsuccessful validation");

This has stopped us from authenticating the user and printed a message to authenticate stopping the user from seeing the data.

This allows us to lockdown our data.

As the application stands at the moment the user can see all other users data and we don't want this to happen so we need to allow the user to see only his own profile. We can do this by changing the route handler.

We know that the **auth** object has authenticated this user and that the response contains the user details so we can rewrite this route handler.

```
    router.get('/users/me', auth, async (req, res) => {
        res.send(req.user);
    });
```

Now, we can test this out again in Postman but first let's rename the request to **Read profile**.

![Postman Read profile](assets/images/postman-read-profile.jpg "Postman Read profile");

Now as you can see it is only sending the authorised users profile information.

### Advanced Postman

Postman has environment variables and we can use these to swap out our localhost variable for our live web address when we deploy our API to Heroku.

#### Local environment

#### Create url variable

We can create a local environment to keep variables for that environment.

We will create a **url** variable with a value of *localhost:3000*.

![Postman Local environment](assets/images/postman-local-environment.jpg "Postman Local environment");

We also create a production environment with no variable as yet.

Now lets work out how to use these environments. Switch over to the **dev** environment and  open up the **Create user** request.

Remove the string *localhost:3000* and replace it with ``{{url}}``. The full request should be `{{url}}/users`

So now we have to change all of our links to have the same url part as above.

We can also do this with our user authentication. In **Read profile**.

1. Remove the token and click on the **Authorization** tab.
2. In the TYPE dropdown select **Bearer Token** and add the token into the **Token** field.
3. Save the request and fire it off again to see that it is working.

This works fine for the one user so now we are going to set this up for all users.

Set TYPE to **Inherit auth from parent**.

Now click on **Edit** Task collection and click on the **Authorization** tab and then select **Bearer Token**. Once again add your token in here. Go back to **Read profile** and try to read the user's profile. If this works you know that you have set it up properly.

Both *Create user* and *Login user* don't require authorization so open both of those tabs up and set **Authorization** to **No Auth**.

All of the other logins are set to **Inherit auth from parent** which is what we require.

#### Create authToken to automate authorization process

We can also automate the one manual task that we do and we can do this with a bit of JavaScript. That is every time we want to authenticate a user we have to add their token into the collection.

Go back to Edit the Task App collection and click on the **Authorization** tab. Remove the token and replace it with ``{{authToken}}``.

Go back to the **Login user** request and click on the **Tests** tab. Type the following code into the edit box.

```
    if (pm.response.code === 200) {
        pm.environment.set('authToken', pm.response.json().token)
    }
```

This will allow us to get the login token for our current user.

To do this you have to login for a particular user and it will automate the token process.

In Postman I use the **Login user** with the following credentials and login.

> "email": "james@james.com.au",  
> "password": "charley@123"  

If I look at the **token** variable for this user at the bottom of the page in Postman I get the following token.

![Postman token variable for the logged in user](assets/images/token-variable.jpg "Postman token variable for the logged in user");

This is the token for ``james@james.com.au`` who is the current logged in user.

Now, if I go to the Environment Management Dialog and look at the Authorization tab I will see that it has the same token saved in the environment variable **authToken**

![Postman Save token in local environment](assets/images/postman-save-token-in-environment.jpg "Postman Save token in local environment");

So now we are going to take the code in the **Tests** tab and add it to the **Create user** Tests tab. We make one small change to the code.

```
    if (pm.response.code === 201) {
        pm.environment.set('authToken', pm.response.json().token)
    }
```

We set the status to 201 which is the code we get back for saving a user and knowing that things went well.

We have now got a completely automated setup and the only thing we have to do is login once and all of the other routes that need authorization will use the token from the current logged in user.

### Logging Out

We are now going to create a way for users to log out of the app. We can do this with a new route.

This route will require authentication to logout.

We have to make a minor addition to **auth.js**

```
    req.token = token;
```

This will save the users current token. Now, all the other handlers will have access to that token and we can use it directly in our new route handler to delete the token off the users **tokens** array.

```
    router.post('/users/logout', auth, async (req, res) => {
        try {
            req.user.tokens = req.user.tokens.filter((token) => {
                return token.token !== req.token;
            });

            await req.user.save();
            res.send();
        } catch (e) {
            res.status(500).send();
        }
    });
```

This will allow us to logout. Now we can create a new **Logout users** request in Postman.

> POST {{url}}/users/logout

Set Authorization to **Inherit auth from parent** and save.

First we need to run the **Read profile** request to make sure a user is logged in.

The current user's token is.

> eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI1ZWY0NDA0MDM1ZjI3NjBjMjg5MGRjNTkiLCJpYXQiOjE1OTMwNjc3MjV9.DSySUzUDLGqMgmAeTr8Y49QmFIlCG-d8yQE2Hw84b9w

We logout the current user and get a status of **200** that lets us know that the user is logged out.

To make sure that the user is really logged out we can go back to the **Read profile** request and click ``Send`` again. This time it shouldn't allow us to see the user's profile.

We should get the following message.

![Postman User not authorised](assets/images/postman-user-not-authorised.jpg "Postman User not authorised");

What if we want to logout of all user sessions?

##### Challenge

Create a new way to logout of all sessions.

1. Setup a route. POST /users/logoutAll
2. Create the route to wipe the tokens array. Send 200 or 500.
3. Test your work Login a few times and logout of all. Check database.

##### LogoutAll route handler

```
    router.post('/users/logoutAll', auth, async (req, res) => {
        try {
            req.user.tokens = [];

            await req.user.save();
            res.send();
        } catch (e) {
            res.status(500).send();
        }
    });
```

Now in Postman create a new request named **LogoutAll users**

> POST {{url}}/users/logoutAll

Set Authorization to **Inherit auth from parent** and save.

First we need to run the **Read profile** request to make sure a user is logged in. There is a user logged in with three tokens.

![Postman Current user with three tokens](assets/images/postman-current-user-with-three-tokens.jpg "Postman Current user with three tokens");

Now we can run the request **LogoutAll users** and we should get a **200**. Now run **Read profile** again and we should get a **401** and the following message

> {  
>     "error": "Please authenticate."  
> }  

Now go to the database and refresh users. You should check for the user who was logged in and they shouldn't have any tokens.

> /* 2 */  
> {  
>     "_id" : ObjectId("5ef4404035f2760c2890dc59"),  
>     "age" : 0,  
>     "name" : "Alan Robson",  
>     "email" : "alan@alan.com.au",  
>     "password" : "$2a$08$cVU0JtQPgVHuCC6OSbe00.cp0BcsRGzQhPJQ4aG3c4wNgOOJFNM4C",  
>     "tokens" : [],  
>     "__v" : 8  
> }  

This means that we have logged the user out of all current logins.

### Hiding Private Data

We need to limit what data gets sent to the client. This will allow us to hide authentication tokens and hashed passwords from server responses.

#### Hiding Private Data

When a Mongoose document is passed to ``res.send``, Mongoose converts the object into JSON. You can customize this by adding ``toJSON`` as a method on the object. The method below removes the **password** and **tokens** properties before sending the response back.

```
    userSchema.methods.toJSON = function () {
        const user = this;
        const userObject = user.toObject();

        delete userObject.password;
        delete userObject.tokens;

        return userObject;
    }
```

Results in.

![Postman Hide current user data](assets/images/postman-hide-data.jpg "Postman Hide current user data");

If I go to the **Read profile** request I can see that I am also hiding data here as well.

![Postman Read profile also hides data](assets/images/postman-read-profile-hide-data.jpg "Postman Read profile also hides data");

### Authenticating User Endpoints

We are now going to add authentication to the other user endpoints.

The first thing we need to do is remove the following route handler.

```
    router.get('/users/:id', async (req, res) => {
        ...
    }
```

This performs the same function as the **Read profile** route handler.

The next route we need to refactor is the **Delete user** route. We need to use authentication on this route. At the moment it is expecting the ``_id`` of a user and we should not allow a user to delete a particular ``_id``. The user should only be allowed to delete themselves.

```
    router.delete('/users/me', auth, async (req, res) => {
        try {
            const user = await User.findByIdAndDelete(req.user._id);

            if (!user) {
                return res.status(404).send();
            }

            res.send(user);
        } catch (e) {
            res.status(500).send();
        }
    });
```

Because we are using **auth** we have access to the current user so we can use ``req.user._id``.

The code above can be refactored ever more because we don't need to check if the user exists in the ``if()`` statement. We also have access to a ``remove()`` method on the Mongoose document. It is asynchronous so we use ``await``.

```
    router.delete('/users/me', auth, async (req, res) => {
        try {
            await req.user.remove();
            res.send(req.user);
        } catch (e) {
            res.status(500).send();
        }
    });
```

Change the Postman request to.

> DELETE /users/me

Make sure Authorization is **Inherit auth from parent** and then you can delete a logged in user.

![Postman Delete authenticated user](assets/images/postman-authenticated-delete.jpg "Postman Delete authenticated user");

##### Challenge

Refactor the Update user route.

1. Update the URL to /users/me
2. Add the Authentication middleware.
3. Use the existing user document instead of fetching the user id.
4. Test your work in Postman.

```
    router.patch('/users/me', auth, async (req, res) => {
        const updates = Object.keys(req.body);
        const allowedUpdates = ['name', 'email', 'password', 'age'];

        const isValidOperation = updates.every((update) => allowedUpdates.includes(update));

        if (!isValidOperation) {
            return res.status(400).send({error: 'Invalid updates.'});
        }

        try {
            updates.forEach((update) => req.user[update] = req.body[update]);

            await req.user.save();

            res.send(req.user);
        } catch (e) {
            res.status(400).send(e);
        }
    });
```

We need to do updates to the **Update user** request in Postman.

> PATCH /users/me

Make sure Authorization is set to *inherit auth from parent*.

### The User/Task Relationship

We need to create a relationship between a user and tasks. This will make it possible to know which tasks a user created.

#### Mongoose Relationships

To set up the relationship, both the user and task model will be changed. First up, a new field needs to be added onto the task. This will store the ID of the user who created it.

##### /src/models/task.js

```
'use strict';

const mongoose = require('mongoose');
const validator = require('validator');

const Task = mongoose.model('Task', {
    description: {
        type: String,
        required: true,
        trim: true
    },
    completed: {
        type: Boolean,
        default: false
    },
    owner: {
        type: mongoose.Schema.Types.ObjectId,
        required: true
    }    
});

module.exports = Task;
```

We add the new field ``owner`` and its type will be a Mongoose **ObjectId** and we need to set ``required`` to true.

Now we need to add authentication to the **Create task** route handler.

```
    const auth = require('../middleware/auth');
```

Now we change the route handler to add the ``owner``.

```
    router.post('/tasks', auth, async (req, res) => {
        const task = new Task({
            ...req.body,
            owner: req.user._id
        });

        try {
            await task.save();
            res.status(201).send(task);
        } catch (e) {
            res.status(400).send(e);
        }
    });
```

Seeing as we have changed the database we need to drop it and restart the server.

Once we do this login with a user and then create a new task. Once you do this you will see the current users **_id** in the ``owner`` field.

![Postman Create task with owner Id](assets/images/postman-create-task-with-owner.jpg "Postman Create task with owner Id");

Now we can expand on this to get the current user values. First, make a small change to our tasks ``owner`` field.

```
    owner: {
        type: mongoose.Schema.Types.ObjectId,
        required: true,
        ref: 'User'
    }
```

We add the ``ref`` value and set it to 'User' model. Now we can create a promise function in **index.js** to get the user object.

```
    const Task = require('./models/task');

    const main = async () => {
        const task = await Task.findById('5ef6cc7de572fc21d02435cc');
        await task.populate('owner').execPopulate();
        console.log(task.owner);
    }

    main();
```

Returns.

> { age: 0,  
>   _id: 5ef6cac296c9c94198cd0b16,  
>   name: 'Alan Robson',  
>   email: 'alan@alan.com.au',  
>   password:  
>    '$2a$08$WaxLtYvwyzc0GUqFn1Hxo.tLwUFDMfZTVL89c13VNHc6aOirqv54.',  
>   tokens:  
>    [ { _id: 5ef6cac296c9c94198cd0b17,  
>        token:  
>         'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI1ZWY2Y2FjMjk2YzljOTQxOThjZDBiMTYiLCJpYXQiOjE1OTMyMzIwNjZ9.> y6At8qIW_Yih32I64x1u6mkMQMkOm7E35IydJCy3xns' } ],  
>   __v: 1 }  

Now we have a relationship between ``User`` and ``Task`` and we can use ``populate`` to get User object.

We have taken a ``Task`` and found a ``User`` from it so now let's go the other way and use a ``User`` to find their tasks.

To do this, a virtual property needs to be added onto the user model. The code below adds a tasks field onto users that can be used to fetch the tasks for a given user. It's a virtual property because users in the database won't have a tasks field. It's a reference to the task data stored in the separate collection.

```
    userSchema.virtual('tasks', {
        ref: 'Task',
        localField: '_id',
        foreignField: 'owner'
    });
```

I change the main function in index to.

```
    const Task = require('./models/task');
    const User = require('./models/user');

    const main = async () => {
        const user = await User.findById('5ef6cac296c9c94198cd0b16');
        await user.populate('tasks').execPopulate();
        console.log(user.tasks);
    }

    main();
```

Returns.

> [ { completed: false,  
>     _id: 5ef6cc57e572fc21d02435ca,  
>     description: 'Take out rubbish',  
>     owner: 5ef6cac296c9c94198cd0b16,  
>     __v: 0 },  
>   { completed: false,  
>     _id: 5ef6cc6ae572fc21d02435cb,  
>     description: 'Clean hall cupboard',  
>     owner: 5ef6cac296c9c94198cd0b16,  
>     __v: 0 },  
>   { completed: false,  
>     _id: 5ef6cc7de572fc21d02435cc,  
>     description: 'Mow the backyard',  
>     owner: 5ef6cac296c9c94198cd0b16,  
>     __v: 0 } ]  

We have created a couple of temporary functions in index.js and now we need to add these to our API.

### Authenticating Task Endpoints

We need to add authentication to our final four task endpoints.

The first task endpoint to change is the *Read a task* by id. We have to make sure they are authenticated and that the task they want to fetch is one that they created.

```
    router.get('/tasks/:id', auth, async (req, res) => {
        const task = await Task.findOne({ _id, owner: req.user._id });

        try {
            const task = await Task.findById(_id);

            if (!task) {
                return res.status(404).send();
            }

            res.send(task);
        } catch (e) {
            res.status(500).send();
        }
    });
```

In the line below both _id's have to  match or the task won't be found.

```
    const task = await Task.findOne({ _id, owner: req.user._id });
```

Test this out in Postman with the **Read task** endpoint.

Grab the last task you created for the currently logged in user.

![Postman Read task created by user](assets/images/postman-current-user-task.jpg "Postman Read task created by user");

The current user can view a task that they created.

Now login as the other user you created and try to run the **Read Task** again. This user didn't create the task so should get a **404**.

![Postman Read task created by other user fails](assets/images/postman-bad-request.jpg "Postman Read task created by other user fails");

The current user can't read a task they didn't create.

##### Challenge

Refactor GET tasks.

1. Add authentication.
2. Return tasks only created by the authenticated user.
3. Test your work.

```
    router.get('/tasks', auth, async (req, res) => {
        try {
            const tasks = await Task.find({ owner: req.user._id });
            res.send(tasks);
        } catch (e) {
            res.status(500).send();
        }
    });
```

or you could do this to get the same results.

```
    router.get('/tasks', auth, async (req, res) => {
        try {
            await req.user.populate('tasks').execPopulate();

            res.send(req.user.tasks);
        } catch (e) {
            res.status(500).send();
        }
    });
```

I am logged in with ``user._id`` 5ef6cac296c9c94198cd0b16.

![Postman Read tasks created by user](assets/images/postman-read-users-tasks.jpg "Postman Read tasks created by user");

Next, we want to update a task by id.

```
    router.patch('/tasks/:id', auth, async (req, res) => {
        const updates = Object.keys(req.body); 
        const allowedUpdates = ['description', 'completed'];

        const isValidOperation = updates.every((update) => allowedUpdates.includes(update));

        if (!isValidOperation) {
            return res.status(400).send({error: 'Invalid updates.'});
        }

        try {
            const task = await Task.findOne({ _id: req.params.id, owner: req.user._id });

            if (!task) {
                return res.status(404).send();

            }

            updates.forEach((update) => task[update] = req.body[update]);

            await task.save();

            res.send(task);
        } catch (e) {
            res.status(400).send(e);
        }
    });
```

We find the task with this line.

```
    const task = await Task.findOne({ _id: req.params.id, owner: req.user._id });
```

And once again we have to be the current user who owns the task.

![Postman User updates a task they created.](assets/images/postman-user-updates-their-task.jpg "Postman User updates a task they created.");

In the case above the user has completed one of the tasks that they created.

##### Challenge

Refactor DELETE /tasks/id.

1. Add authentication.
2. Find the task by _id/owner (findOneAndDelete).
3. Test your work.

```
    router.delete('/tasks/:id', auth, async (req, res) => {
        const _id = req.params.id;

        try {
            const task = await Task.findOneAndDelete({ _id, owner: req.user._id });

            if (!task) {
                return res.status(404).send();
            }

            res.send(task);
        } catch (e) {
            res.status(500).send();
        }
    });
```

I logged in as me with an _id of 5ef6cac296c9c94198cd0b16.

![Postman User deletes a task they created.](assets/images/postman-delete-user-task.jpg "Postman User deletes a task they created.");

I was able to successfully delete a task that I had created.

I now try to delete a task created by another user _id. I end up getting a 404 error as I expected.

### Cascade Delete Tasks

We will now use Mongoose middleware to clean up a user's tasks when they close their account. This will make sure that all their data is securely removed from the database.

#### Deleting a User's Tasks

The middleware function below is registered using **pre**. It will run just before ``remove`` fires for the user. The function itself deletes all tasks created by that user. Now when a user closes their account, their other data will get deleted too.

```
    const Task = require('./task');

    userSchema.pre('remove', async function (next) {
        const user = this;

        await Task.deleteMany({ owner: user._id});

        next();
    });
```
