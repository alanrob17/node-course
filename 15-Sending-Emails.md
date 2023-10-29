## Sending Emails

### Section Intro

We can add email sending to your Node.js application! This will allow you to communicate with users as they use the app. This could be useful for welcome emails, notifications, and more!

### Exploring SendGrid

We will integrate SendGrid into our Node app. SendGrid is one of many services that allow you to send emails from your application code.

First up, install the module.

```
    npm i sendgrid/mail
```

Next, create a free SendGrid account and get your API key. The code below shows what's necessary to get the SendGrid module configured. All you need to do is call setApiKey and set your API key.

##### emails/account.js

```
    const sgMail = require('@sendgrid/mail');

    const sendgridApiKey = '';

    sgMail.setApiKey(sendgridApiKey);

    sgMail.send({
        to: 'alan@robsonmail.com',
        from: 'alan@robsonmail.com',
        subject: 'This is my first email!',
        text: 'I hope this one actually gets to you.'
    });
```

we could create HTML in the text field for formatting if required.

In the long term, you'll want to purchase a custom domain and register it with SendGrid. This will increase your sending reliability.

Send a welcome email to a newly created user.

##### src/routers/user.js

```
    const { sendWelcomeEmail, sendCancelationEmail } = require('../emails/account');

    // POST /users/user details
    router.post('/users', async (req, res) => {
        const user = new User(req.body);

        try {
            await user.save();
            sendWelcomeEmail(user.email, user.name);
            const token = await user.generateAuthToken();

            res.status(201).send({ user, token });  
        } catch (e) {
            res.status(400).send(e);  
        }
    });
```

Send an email to a user who is unsubscribing.

##### src/routers/user.js

```
    // DELETE /users/me
    router.delete('/users/me', auth, async (req, res) => {
        try {
            await req.user.remove();
            sendCancelationEmail(req.user.email, req.user.name);
            res.send(req.user);
        } catch (e) {
            res.status(500).send();
        }
    });
```

### Environment Variables

We need to learn how to use environment variables to securely store API keys and other credentials. This will reduce the chance your private keys fall into the wrong hands.

First up, install the npm module. We will only need this on our dev environment and not when we deploy to Heroku.

```
    npm i env-cmd --save-dev
```

Next up, create an environment file ``dev.env`` in the **config** directory. This will store your environment variables in the following format.

```
    KEY=value
    ANOTHER_KEY=some other value
```

Next, update the **dev** script to use env-cmd to load in those environment variables when it starts up. That would be **env-cmd -f ./config/dev.env nodemon src/index.js**.

##### dev.env

```
    PORT=3000
    SENDGRID_API_KEY=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
    MONGODB_URL=mongodb://127.0.0.1:27017/task-manager-api
    JWT_SECRET=thisismynodecourse
```

Now, you can remove API keys and database credentials from the application itself. For example, you can create **MONGODB_URL** in the development environment file. The application code shown below can then reference that environment variable to get its value. This can be done with the SendGrid API key and the JWT secret used to generate and verify authentication tokens.

##### src/db/mongoose.js
```
    mongoose.connect(process.env.MONGODB_URL, {
        useNewUrlParser: true,
        useUnifiedTopology: true,
        useCreateIndex: true,
        useFindAndModify: false
    });
```

##### src/middleware/auth.js

```
    const auth = async (req, res, next) => {
       try {
        const token = req.header('Authorization').replace('Bearer ', '');
        const decoded = jwt.verify(token, process.env.JWT_SECRET);
        const user = await User.findOne({ _id: decoded._id, 'tokens.token': token });

        if (!user) {
            throw new Error();
        }

        req.token = token;
        req.user = user;

        next();
       } catch (e) {
           res.status(401).send({ error: 'Please authenticate.'});
       }
    };
```

##### src/index.js

```
    const port = process.env.PORT;
```

##### src/emails/account.js

```
    sgMail.setApiKey(process.env.SENDGRID_API_KEY);
```

### Creating a Production MongoDB Database

We need to set up a production MongoDB database using MongoDB Atlas. Atlas is the official MongoDB hosting platform released by the MongoDB organization.

We create a free cluster for our **Production** database.

Use AWS to serve our Database.

![Use AWS to host our database](assets/images/aws-cluster.jpg "Use AWS to host our database")

More settings.

![Additional settings](assets/images/additional-settings.jpg "Additional settings")

We need to use the IP address 0.0.0.0/0 which allows us to use all IP's for our database.

![Use this IP address](assets/images/ip-address.jpg "Use this IP address")

Create a username and password

```
    U: taskapp
    P: password
```

Connect using MongoDB Compass.

Use this connection string to connect to your cluster.

```
    mongodb+srv://taskapp:<password-here>@cluster0.e9h9h.mongodb.net/test
```

In the string above **test** is the database name and we have to change this to **task-manager-api**.

I have saved this as a favourite **Production Mongo DB** so I can open it quickly.

I have also created a localhost connection and that shows the Task App database. I have saved this as a favourite, **localhost Mongo DB**.

### Heroku Deployment

We are now ready to deploy the Task manager API to Heroku.

We need to do some tasks on our application so we can deploy it.

Add the Task manager to Git.

```
    git init
```

Create a ``.gitignore``.

```
    node_modules
    config
```

Run the commands.

```
    git add .

    git commit -m "Initial commit."
```

Now we can create a new repository on Github and note that you are able to create private repositories for free.

We can run the next commands to upload the local repository to Github. We will call it **task-manager-api**

```
    git remote add origin https://github.com/alanrob17/task-manager-api.git

    git push -u origin master
```

Now you can create your Heroku app

**Note** Since my new machine build I have lost the Heroku application on my PC so I had to download and install the 64-bit installer on my machine. Once I did this I had trouble creating a new app until I was pushed up to Heroku and it logged me in. Once it did this I was able to create the new application.

```
    heroku create alanr-task-manager
```

Once the application was created it linked my Github repository to my Heroku application.

```
    https://alanr-task-manager.herokuapp.com/ | https://git.heroku.com/alanr-task-manager.git
```

This is the url to my new application.

We now need to setup some environment variables for our application.

We already know how to deploy Node.js applications to Heroku. The only difference with this application is that your custom environment variables need to be configured on Heroku too. This doesn't include PORT, as that's managed by Heroku.

The command below is used to set an environment variable.

```
    heroku config:set KEY=VALUE
```

A variation of that command can be used to fetch all the environment variables currently configured.

```
    heroku config
```

Lastly, you can delete an environment variable as shown here.

```
    heroku config:unset KEY
```

Go back to the Cloud MongoDB cluster and click **connect**.

![Connect](assets/images/connect.jpg "Connect")

Click on **connect your application**.

![Connect to your application](assets/images/connect-your-application.jpg "connect to your application")

Add the password and the database name to the MONGODB_URL line.

Run each key in the VS Code terminal.

```
    heroku config:set JWT_SECRET=thisismynodecourse
    heroku config:set SENDGRID_API_KEY=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
    heroku config:set MONGODB_URL='mongodb+srv://taskapp:<password here>@cluster0.e9h9h.mongodb.net/task-manager-api?retryWrites=true&w=majority'
```

Once you are sure these keys have been uploaded to Heroku.

```
    heroku config
```

Now push your code to Heroku.

```
    git push heroku master
```

Your new API URL is shown in the terminal.

https://alanr-task-manager.herokuapp.com/

We can use this in Postman so see if we can connect to our API.

In Postman open up your production environment and add the URL.

![Postman production environment](assets/images/postman-prod-environment.jpg "Postman production environment")

**NOTE:** don't forget to remove the closing forward slash of the URL.

Click Update and in Postman select the Production Environment and we are ready to test by selecting the **Read task** request. You should get a *Please authenticate* message back because we haven't created a user. This shows that our application is working.

So now create a new **user**.

Postman should work exactly the same as it did in the development environment. You should be able to create tasks and view these tasks.
