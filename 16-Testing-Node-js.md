## Testing Node.js

### Section Intro

We are going to learn how to test our Node.js applications. Setting up an automated test suite makes it easy to check that your application is always working as expected.

### Jest Testing Framework

We will be using the Jest testing framework. Jest gives you everything you need to create a test suite for your Node.js applications.

### Setting up Jest

Install the module.

```
    npm i jest
```

Next, create a test script in package.json. The script itself is jest. This allows you to use **npm test** to run the Jest test suite.

Now, you'll need to create a test suite. This is a file in your project that ends with **.test.js**. The file extension allows Jest to find and run the test suites for your project.

### Creating a Test Case

You can add a test case to a test suite using the test function. Jest provides various functions as global variables in your test suite files. test is one of them. The first argument to test is the name of your test case. The second argument to test is the test function itself.

If the test function throws an error, the test cast will fail. If the test function doesn't throw an error, the test case will pass.

The test case below would always pass, as no error is thrown.

```
    test('Hello world!', () => {
    });
```

The test case below would always fail, as it throws an error.

```
    test('This should fail', () => {
        throw new Error('Failure!');
    });
```

I can now run a test by.

```
    npm test
```

These are the results.

![Jest test results 1](assets/images/jest-test-01.jpg "Jest test results 1")

### Writing Tests and Assertions

We can add assertions to our test cases. Assertions allow you to check if a given value is what you're expecting or not.

Create our function.

##### src/math.js

```
    const calculateTip = (total, tipPercent) => {
        const tip = total * tipPercent;
        return total + tip;
    }

    module.exports = { calculateTip }
```

Create our test.

##### tests/math.test.js

```
    const { calculateTip } = require('../src/math');

    test('Should calculate total with tip.', () => {
        const total = calculateTip(10, .3);

        if (total !== 13) {
            throw new Error('Total tip should be 13. Got ' + total);
        }
    });
```

When I run the test I get.

![Successful test](assets/images/successful-test.jpg "Successful test")

Things ran as expected so we are getting a good result. Imagine when we wrote the original function we used a minus instead of a plus to calculate the total.

```
    const calculateTip = (total, tipPercent) => {
        const tip = total * tipPercent;
        return total - tip;
    }
```

We don't have to change our test as it is set up correctly.

When we run the test again.

![Calculate tip error](assets/images/calculate-tip-error.jpg "Calculate tip error")

Now we see that we have an error in **math.js**. We got an incorrect result. Should have been 13 but we got 7 so we can go back to the calculateTip function and try and find where our error is occuring. Once we have found the error do the test again to make sure it is working correctly.

The way we wrote our test can be improved.

The test suite below has a single test case for the **calculateTip** function. The test case itself calculates a 30% tip on a $10 restaurant bill. The assertion checks that the calculated total equals $13. The assertion is made using ``toBe`` to check for equality.

```
    const { calculateTip } = require('../src/math');

    test('Should calculate total with tip', () => {
        const total = calculateTip(10, .3);
        expect(total).toBe(13);
    });
```

Add the error back into the function and run the test again.

![Expect error](assets/images/expect-error.jpg "Expect error")

This is a more efficient way of writing your test.

Now we can refactor our function knowing the test is in place and doesn't have to be rewritten.

```
    const calculateTip = (total, tipPercent) => total + (total * tipPercent);
```

Rerun the test to make sure we haven't broken anything.

We can also add a default value for the tip in case the user forgets to add a tip value.

```
    const calculateTip = (total, tipPercent = .25) => total + (total * tipPercent);
```

In this case we have to write another test to catch the default tip value.

Add this test.

```
    test('Should calculate the total with the default tip.', () => {
        const total = calculateTip(10);

        expect(total).toBe(12.5);
    });
```

When we run the tests we get the following results.

![Default value test](assets/images/default-value-test.jpg "Default value test")

In this case both tests run correctly.

##### Challenge

Test temperature conversion functions

1. Export both functions and load them into the test suite.
2. Create "Should convert 32 F to 0 C".
3. Create "Should convert 0 C to 32 F".
4. Run the Jest to test your work!

```
    test('Should convert 32 F to 0 C.', () => {
        const celsius = fahrenheitToCelsius(32);

        expect(celsius).toBe(0);
    });

    test('Should convert 0 C to 32 F.', () => {
        const fahrenheit = celsiusToFahrenheit(0);

        expect(fahrenheit).toBe(32);
    });
```

Returns the following results.

![Temperature tests passed](assets/images/four-tests-passed.jpg "Temperature tests passed")

Once again I can refactor both functions to short arrow functions

```
    const fahrenheitToCelsius = (temp) => (temp - 32) / 1.8;

    const celsiusToFahrenheit = (temp) => (temp * 1.8) + 32;
```

I then rerun all of my tests to make sure I haven't broken my functions and I should get the same results as above.

We can set Jest up to run every time our code changes.

##### package.json (partial)

```
        "dev": "env-cmd -f ./config/dev.env nodemon src/index.js",
        "test": "jest --watch"
      },
      "dependencies": {
```

### Testing Asynchronous Code

We need to test asynchronous code. This will be necessary to test the Express API endpoints.

let's create a ``setTimeout()`` callback function.

```
    test('Async test demo.', () => {
        setTimeout(() => {
            expect(1).toBe(2);
        }, 2000);
    });
```

When the tests run they will all pass. This is clearly incorrect. Jest won't wait for the test to run after 2 seconds so the ``expect()`` test runs after Jest has completed its tests.

We can force Jest to wait by calling a function in the callback.

```
    test('Async test demo.', (done) => {
        setTimeout(() => {
            expect(1).toBe(2);
            done();
        }, 2000);
    });
```

![setTimeout test failure](assets/images/settimeout-failure.jpg "setTimeout test failure")

The two test cases below test the asynchronous **add** function you created earlier in the course. Both test cases add up 2 and 3 and assert that the total is 5.

```
    const add = (a, b) => {
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                if (a < 0 || b < 0) {
                    return reject('Numbers must be non-negative');
                }

                resolve(a + b);
            }, 2000);
        });
    }
```

The callback function for the first test case accepts a ``done`` parameter. This lets Jest know that the test function contains asynchronous code. Jest won't determine if the test passed or failed until ``done`` is called. In the example below, ``then`` is called to run some code after the numbers are added. This is where the assertion is added and it's where ``done`` is called.

```
    test('Should add two numbers.', (done) => {
        add(2, 3).then((sum) => {
            expect(sum).toBe(5);
            done();
        });
    });
```

Your test cases can use **async/await** as well. The test case below is a refactored version of the test case above. The test case function is defined with ``async``. ``await`` is used in the function to ensure that Jest waits for those asynchronous tasks to complete. Both test cases are functionally identical.

```
    test('Should add two numbers async/await.', async () => {
        const sum = await add(10, 22);

        expect(sum).toBe(32);
    });
```

This works because Jest will wait for a promise to finish. We don't have to use the dummy ``done()`` function.

Both of the tests above perform the same task but it is preferable to use the **async/await** syntax.

### Testing an Express Application: Part I

We are going to set up the Express API to be easily testable. This involves settings up a test environment as well as configuring Jest to work with Node.

#### Creating a Test Environment

Creating the test environment requires **test.env** to be added to the **config** directory. The contents will be identical to **dev.env**, with the exception of the MongoDB connection string. The test environment should use a separate database such as **task-manager-api-test**. This will prevent the test cases from messing with development data.

##### test.env

```
PORT=3000
SENDGRID_API_KEY=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
MONGODB_URL=mongodb://127.0.0.1:27017/task-manager-api-test
JWT_SECRET=thisismynodecourse
```

We have only made one change to our test.env from our dev.env variables and that is a different database name.

Why don't we just use the development environment for our testing? We don't want to fill our development database with seed data so we need to create a new test database.

With the environment in place, update the test script to load the environment file in. That would be **env-cmd .f ./config/test.env jest --watch --runInBand**.

#### Configuring Jest

By default, Jest is expecting to run in the browser. You can use Jest with Node, but you'll need to configure Jest to enable support. Jest can be configured by adding a **jest** property in package.json. The configuration below sets testEnv to node to ensure that Jest runs correctly in Node.js.

```
{
    "jest": {
        "testEnv": "node"
    }
}
```

For more information check out [configuring Jest.](https://jestjs.io/docs/en/configuration "configuring Jest.")

### Testing an Express Application: Part II

We are now ready to add tests for the Express API. Each test case will focus on testing a specific endpoint, making assertions about the response from the server.

#### Testing with Supertest

Supertest was created by the Express team to allow you to easily test your Express apps.

First up, install the module.

```
    npm i supertest --save-dev
```

We also need to make changes to our application to allow it to work in the test environment. Our test app doesn't need to run the express application so we don't need to use ``app.listen()``. We are going to set up an **app.js** file to run our test environment and will leave our **index.js** file for development and production.

##### app.j

```
    const express = require('express');
    const userRouter = require('./routers/user');
    const taskRouter = require('./routers/task');
    require('./db/mongoose');

    const app = express();

    app.use(express.json());
    app.use(userRouter);
    app.use(taskRouter);

    module.exports = app;
```

We have take most of the code from **index.js** and will call **app.js** in **index.js** to stop us from repeating a lot of the code.

##### index.js

```
    'use strict';

    const app = require('./app');

    const port = process.env.PORT;

    app.listen(port, () => {
        console.log('Server is up on port ' + port);
    });
```

Now, supertest can be used to test an endpoint. The test case below tests that new users can sign up for accounts. All the account data provided is valid, so a new account should be created.

Step one is to pass the express **app** to **request**. Next, supertest methods can be chained together to fit the needs of your tests. **post** is used to make an HTTP POST request to **/users**. **send** is used to send the correct JSON data to the server. **expect** is used to assert that the response status code is correct. In this case, a successful signup should result in a **201** status code.

##### /tests/user.test.js

```
    const request = require('supertest');

    const app = require('../src/app');

    test('Should sign up a new user.', async () => {
        await request(app).post('/users').send({
            name: 'Alan',
            email: 'alan@robsonmail.com',
            password: 'alan@123'
        }).expect(201);
    });
```

Results in.

![New user test passed](assets/images/user-test-success.jpg "New user test passed")

In the database (Robo 3T).

![New user in the test database](assets/images/new-user-in-test-db.jpg "New user in the test database")

In the MongoDB database

![New user in the MongoDB test database](assets/images/mongo-db-new-user.jpg "New user in the MongoDB test database")

There is a minor problem with our code. If I rerun the test (type ``a`` in the terminal kicks it off again) I get a **401** bad request because the user record is already in the database.

In this case we need to tell Jest to wipe the database before we run our tests again.

### Jest Setup and Teardown

We need to configure Jest to add test data into the database. This will allow you to test operations that require existing data, such as the login operation. You can't log in if there isn't a user account in the database to login to.

#### Seeding Database

Jest provides lifecycle functions that you can use to configure your test suite. There are four:

1. beforeEach - Run some code before each test case
2. afterEach - Run some code after each test case
3. before - Run some code once before the tests run
4. after - Run some code once after the tests run

``beforeEach`` works great for adding test data to the database. The ``beforeEach`` call below removes all users and then adds a single test user into the database. By having this run before each test case, it ensures that the tests run in a consistent environment each time they execute.

We add this code before our tests to delete all users in the test database.

```
    const User = require('../src/models/user');

    beforeEach(async () => {
        await User.deleteMany();
    });
```

Once again we are using promises to make sure all users are deleted before we continue on to the tests.

This time when we run the tests we will see that they are passing again.

What if we need to add a user for particular tests? We can add one after we delete all users in ``beforeEach()``. This can then be used for any tests ,like login() that require a user to be available for testing.

##### Challenge

Test login failure

1. Create "Should not login non-existent user".
2. Send off the request with bad credentials.
3. Expect the correct status response.
4. Test your work.

```
    test('Should not login non-existent user.', async () => {
        await request(app).post('/users/login').send({
            email: 'fred@fred.com',
            password: 'doggo@1'
        }).expect(400);
    });
```

Results.

![Can't login non existent user](assets/images/cant-login-non-existent-user.jpg "Can't login non existent user")

##### Challenge

Test Delete account.

1. Create "Should delete account for user" - Setup auth header and expect correct status code.
2. Create "Should not delete account for unauthorised user". - expect correct status code.
3. Test your work

```
    test('Should delete account for user.', async() => {
        await request(app)
            .delete('/users/me')
            .set('Authorization', `Bearer ${userOne.tokens[0].token}`)
            .send()
            .expect(200);
    });

    test('Should not delete account for unauthorised user.', async() => {
        await request(app)
            .delete('/users/me')
            .send()
            .expect(401);
    });
```

These tests pass.

### Advanced Assertions

We can test the information that is being created when we test creating a new user. We can create a variable for the response and test what was sent back. In the case below we are testing to see that the user has a name value.

```
    test('Should sign up a new user.', async () => {
        const response = await request(app).post('/users').send({
            name: 'Alan',
            email: 'alan@robsonmail.com',
            password: 'alan@123'
        }).expect(201);

        // Assert that the user was changed correctly.
        const user = await User.findById(response.body.user._id);
        expect(user).not.toBeNull();

        // Assertions about the response.
        expect(response.body.user.name).toBe('Alan');
    });
```

We can test the whole object sent back if we need to.

```
    test('Should sign up a new user.', async () => {
        const response = await request(app).post('/users').send({
            name: 'Alan',
            email: 'alan@robsonmail.com',
            password: 'alan@123'
        }).expect(201);

        // Assert that the user was changed correctly.
        const user = await User.findById(response.body.user._id);
        expect(user).not.toBeNull();

        // Assertions about the response.
        expect(response.body).toMatchObject({
            user: {
                name: 'Alan',
                email: 'alan@robsonmail.com'
            },
            token: user.tokens[0].token
        });

        expect(user.password).not.toBe('alan@123');
    });
```

##### Challenge

Validate new token is being saved.

1. Fetch the user from the database.
2. Assert that the token in the response matches the users second token.
3. Test your work.

```
    test('Should login an existing user.', async () => {
        const response = await request(app).post('/users/login').send({
            email: userOne.email,
            password: userOne.password
        }).expect(200);

        const user = await User.findById(response.body.user._id);

        expect(response.body.token).toBe(user.tokens[1].token);
    });
```

Tests pass.

### Challenge

Validate user is removed.

1. Fetch the user from the database.
2. Assert null response (use assertion from the signup test).
3. Test your work.

```
    test('Should delete account for user.', async() => {
        const response = await request(app)
            .delete('/users/me')
            .set('Authorization', `Bearer ${userOne.tokens[0].token}`)
            .send()
            .expect(200);

        const user = await User.findById(userOneId);
        expect(user).toBeNull();
    });
```

Tests pass.

### Mocking Libraries

We are now going to learn how to mock npm modules. Jest lets you mock npm modules so you can override module functionality in your test environment.

### Mocking SendGrid

You can mock an npm module by creating a **\_\_mocks\_\_** directory in the tests folder. A module can be mocked by creating a file in the **\_\_mocks\_\_** folder. The file name should match up with the module name, so **tests\/\_\_mocks\_\_\/express.js** can be used to mock the Express library. If the npm module uses a scope like **@sendgrid/mail**, then a **@sendgrid** folder would be created with a **mail.js** file inside.

The job of the mock file is to provide mocked versions of the library features. The SendGrid mock below defines and exports **setApiKey** and **send**. This ensures that our code still works even though emails will no longer be sent for the tests.

##### /src/tests/\_\_mocks\_\_/@sendgrid/mail.js

```
    module.exports = {
        setApiKey() {

        },
        send() {

        }
    }
```

### Wrapping up User Tests

We need to create a test to upload an avatar. To do this we create a folder named **fixtures** in **tests**. This is where we put assets we need for our testing.

We will put our profile picture in this folder.

```
    test('Should upload avatar image.', async() => {
        await request(app)
            .post('/users/me/avatar')
            .set('Authorization', `Bearer ${userOne.tokens[0].token}`)
            .attach('avatar', 'tests/fixtures/profile-pic.jpg')
            .expect(200);
    });
```

![Test upload avatar](assets/images/test-avatar.jpg "Test upload avatar")

We want to carry out a further test to see if the user has an avatar.

```
    test('Should upload avatar image.', async() => {
        await request(app)
            .post('/users/me/avatar')
            .set('Authorization', `Bearer ${userOne.tokens[0].token}`)
            .attach('avatar', 'tests/fixtures/profile-pic.jpg')
            .expect(200);

        const user = await User.findById(userOneId);
        expect(user.avatar).toEqual(expect.any(Buffer));
    });
```

**Note:** we are using ``toEqual()`` instead of ``toBe()``. ``toBe()`` does a strict equality test and because we are checking an object the strict equality test would fail. This is where we have to use ``toEqual`` and what we are testing is to see that the field ``user.avatar`` contains a buffer. As long as there is a buffer it means that we have uploaded an image and stored it in ``user.avatar``.

Once we save we see that our tests have passed.

##### Challenge

Test user updates.

1. Create "Should update valid user fields". - Update the name of the test user. - Check the data to confirm it has changed.
2. Create "Should not update invalid user fields". - Update a "location" field and expect a status error code.
3. test your work.

##### Test 1.

```
    test('Should update valid user fields.', async() => {
        await request(app)
            .patch('/users/me')
            .set('Authorization', `Bearer ${userOne.tokens[0].token}`)
            .send({
                name: 'Charley'
            })
            .expect(200);

        const user = await User.findById(userOneId);
        expect(user.name).toEqual('Charley');
    });
```

##### Test 2.

```
    test('Should not update invalid user fields.', async() => {
        await request(app)
            .patch('/users/me')
            .set('Authorization', `Bearer ${userOne.tokens[0].token}`)
            .send({
                name: 'James',
                location: 'Melbourne, Vic'
            })
            .expect(400);
    });
```

### Setup Task Test Suite

Before we can set up the Task tests we need to change the setup of our testing code.

##### tests/fixtures/db.js

```
    const jwt = require('jsonwebtoken');
    const mongoose = require('mongoose');
    const User = require('../../src/models/user');

    const userOneId = new mongoose.Types.ObjectId();

    const userOne = {
        _id: userOneId,
        name: 'James',
        email: 'james@robsonmail.com',
        password: 'james@123',
        tokens: [{
            token: jwt.sign({ _id: userOneId }, process.env.JWT_SECRET)
        }]
    }

    const setupDatabase = async () => {
        await User.deleteMany();
        await new User(userOne).save();
    }

    module.exports = {
        userOneId,
        userOne,
        setupDatabase
    }
```

The **db.js** file takes out code from the **user.test.js** files so we can use our new user in our **task.test.js** file. This makes our test user available to both sets of tests.

##### tests/user.test.js (setup)

```
    const request = require('supertest');

    const app = require('../src/app');
    const User = require('../src/models/user');

    const { userOneId, userOne, setupDatabase } = require('./fixtures/db');

    beforeEach(setupDatabase);
```

##### tests/task.test.js (setup)

```
    const request = require('supertest');
    const app = require('../src/app');
    const Task = require('../src/models/task');
    const { userOneId, userOne, setupDatabase } = require('./fixtures/db');

    beforeEach(setupDatabase);
```

Before we can get our tests to work correctly we have to make a change to **package.json**

```
    "test": "env-cmd -f ./config/test.env jest --watch --runInBand"
```

The setting **--runInBand** will make sure our tests run in series and that there is no overlap.

Now we can write our first task test.

```
    test('Should fetch user tasks', async () => {
        const response = await request(app)
            .post('/tasks')
            .set('Authorization', `Bearer ${userOne.tokens[0].token}`)
            .send({
                description: 'Play fortnite'
            })
            .expect(201);

        const task = await Task.findById(response.body._id);
        expect(task).not.toBeNull();
        expect(task.completed).toEqual(false);
    });
```

In the test above we can create a new task and then test that it is not null and also test that the completed field is **false**.

### Testing with Task Data

Testing tasks will require that some test tasks exist in the database. Like with users, tasks can be added to the database using ``beforeEach``. The test case below fetches all tasks for the user. It also asserts that the status code is a **200** and the tasks are sent back correctly.

##### Challenge

Test delete Task security (one user should not be able to delete another user's tasks).

1. Attempt to have the second user delete the first task (should fail). - setup necessary exports from db.js.
2. Assert the failure status code.
3. Assert the first task is still in the database.
4. Test your work.

```
    test('Should not delete other users tasks.', async () => {
        const response = await request(app)
            .delete(`/tasks/${taskOne._id}`)
            .set('Authorization', `Bearer ${userTwo.tokens[0].token}`)
            .send()
            .expect(404);

            const task = await Task.findById(taskOne._id);
            expect(task).not.toBeNull();
    });
```

#### More user tests

The following lists are extra tests you could create.

```
    User Test Ideas

        Should not signup user with invalid name/email/password
        Should not update user if unauthenticated
        Should not update user with invalid name/email/password
        Should not delete user if unauthenticated

    Task Test Ideas

        Should not create task with invalid description/completed
        Should not update task with invalid description/completed
        Should delete user task
        Should not delete task if unauthenticated
        Should not update other users task
        Should fetch user task by id
        Should not fetch user task by id if unauthenticated
        Should not fetch other users task by id
        Should fetch only completed tasks
        Should fetch only incomplete tasks
        Should sort tasks by description/completed/createdAt/updatedAt
        Should fetch page of tasks
```