## Asynchronous Node.js

### Section Intro

It's time to connect our application with the outside world. In this section, we'll explore the asynchronous nature of Node.js. We'll learn how to use asynchronous programming to make HTTP API requests to third-party HTTP APIs. This will allow you to pull in data, like real-time weather data, into your app.

### Asynchronous Basics

When running asynchronous code, your code won't always execute in the order you might expect. To get started with asynchronous development, let's use setTimeout. setTimeout is a function that allows you to run some code after a specific amount of time has passed. setTimeout accepts two arguments. The first is a callback function. This function will run after the specified amount of time has passed. The second argument is the amount of time in milliseconds to wait.

```
    console.log('Starting')

    // Wait 2 seconds before running the function
    setTimeout(() => {
        console.log('2 Second Timer');
    }, 2000);

    console.log('Stopping');
```

Run the script and you'll see the logs in the following order.

> Starting
> Stopping
> 2 Second Timer

Notice that "Stopping" prints before "2 Second Timer". That's because **setTimeout** is asynchronous and non-blocking. The **setTimeout** call doesn't block Node.js from running other code while it's waiting for the 2 seconds to pass.

This asynchronous and non-blocking nature makes Node.js ideal for backend development. Your server can wait for data from a database while also processing an incoming HTTP request.

### Making HTTP Requests

We are going to learn how to make HTTP requests from Node. This will enable your app to communicate with other APIs and  servers to do a wide variety of things. Everything from fetching real-time weather data to sending text messages to users.

There are several libraries that make it easy to fire off HTTP requests. We will use **postman-request**. You can install it using the command below.

```
    npm i postman-request
```

Before you use the library in your app, you'll need to figure out which URL you're trying to fetch. To fetch real-time weather data, you'll need to sign up for a free Webstack API account.

Below is an example URL

```
    const request = require('postman-request');

    const url = 'http://api.weatherstack.com/current?access_key=f41e35407998bf659423ffd2ac284503&query=-37.6158588,143.7173154& units=m';

    request({url: url}, (error, response) => {
        console.log(response);
    });
```

This request brings back way too much information but right at the bottom you see the weather JSON.

We can refine this code.

```
    const request = require('postman-request');

    const url = 'http://api.weatherstack.com/current?access_key=f41e35407998bf659423ffd2ac284503&query=-37.6158588,143.7173154& units=m';

    request({url: url}, (error, response) => {
        const data = JSON.parse(response.body);

        console.log(data);
    });
```

We can refine this further with

```
    console.log(data.current);
```

##### Results

```
    {
      "request": {
        "type": "LatLon",
        "query": "Lat -37.62 and Lon 143.72",
        "language": "en",
        "unit": "m"
      },
      "location": {
        "name": "Nintingbool",
        "country": "Australia",
        "region": "Victoria",
        "lat": "-37.617",
        "lon": "143.717",
        "timezone_id": "Australia/Melbourne",
        "localtime": "2020-06-06 12:54",
        "localtime_epoch": 1591448040,
        "utc_offset": "10.0"
      },
      "current": {
        "observation_time": "02:54 AM",
        "temperature": 7,
        "weather_code": 143,
        "weather_icons": [
          "https://assets.weatherstack.com/images/wsymbols01_png_64/wsymbol_0006_mist.png"
        ],
        "weather_descriptions": [
          "Mist"
        ],
        "wind_speed": 1,
        "wind_degree": 181,
        "wind_dir": "S",
        "pressure": 1027,
        "precip": 0,
        "humidity": 86,
        "cloudcover": 29,
        "feelslike": 7,
        "uv_index": 2,
        "visibility": 7,
        "is_day": "yes"
      }
    }
```

This is another example using Open Weathermap.

```
    const unirest = require("unirest");

    const req = unirest("GET", "https://community-open-weather-map.p.rapidapi.com/weather");

    req.query({
    	"callback": "test",
    	"id": "2172797",
    	"units": "metric",
    	"mode": "xml%2C html",
    	"q": "Haddon,au"
    });

    req.headers({
    	"x-rapidapi-host": "community-open-weather-map.p.rapidapi.com",
    	"x-rapidapi-key": "9cb79bc01cmsh36974c8d075a759p1b66c1jsn5c64f069e269",
    	"useQueryString": true
    });

    req.end(function (res) {
    	if (res.error) throw new Error(res.error);

    	console.log(res.body);
    });
```

### Customizing HTTP Requests

#### Request Options

The request library comes with plenty of options to make your life easier. One is the json option. Set json to true and request will automatically parse the JSON into a JavaScript object for you.

```
    const request = require('postman-request');

    const url = 'http://api.weatherstack.com/current?access_key=f41e35407998bf659423ffd2ac284503&query=-37.6158588,143.7173154& units=m';

    request({url: url, json: true}, (error, response) => {
        console.log(response.body.current);
    });
```

##### Challenge

Print a small forecast for the user.

1. Print "It is currently 7 degrees out. There is 0% chance of rain."
2. Test your work.

```
    const request = require('postman-request');

    const url = 'http://api.weatherstack.com/current?access_key=f41e35407998bf659423ffd2ac284503&query=-37.6158588,143.7173154& units=m';

    request({url: url, json: true}, (error, response) => {
        const temperature = response.body.current.temperature;
        const precipitation = response.body.current.precip;
        const description = response.body.current.weather_descriptions;
        console.log(`It is currently ${temperature} degrees out. It is ${description} and there is ${precipitation}% chance of  rain.`);
    });
```

> It is currently 10 degrees out. It is Overcast and there is 0% chance of rain.

### An HTTP Request Challenge

#### Geocoding

We use the mapbox.com api to forward geocode.

```
    const locationUrl = 'https://api.mapbox.com/geocoding/v5/mapbox.places/melbourne.json?access_token=pk.  eyJ1IjoiYWxhbnJvYjE3IiwiYSI6ImNrYjM0cWJmaDBoaXUycXFwNWhmc21yZHoifQ.fmtQdmsoTDNr6JWInac8kA';

    request({url: locationUrl, json: true}, (error, response) => {
        const location = response.body.features[0].place_name;
        const long = response.body.features[0].center[0];
        const lat = response.body.features[0].center[1];

        console.log(`The town is ${location}. Its latitude is ${lat} and longitude is ${long}.`);
    });
```

> The town is Melbourne, Victoria, Australia. Its latitude is -37.8142 and longitude is 144.9632.

### Handling Errors

There are plenty of reasons an HTTP request can fail. Maybe your machine doesn't have an internet connection, or maybe the URL is incorrect. Regardless of what goes wrong, you need to be able to handle errors that occur when making HTTP requests.

Handling errors is important. It would be nice if we could always provide the user with a forecast for their location, but that's not going to happen. When things fail, you should aim to provide users with clear and useful messages in plain English so they know what's going on.

The callback function you pass to request expects an **error** and **response** argument to be provided. Either **error** or **response** will have a value, never both. If **error** has a value, that means things went wrong. In this case, **response** will be undefined, as there is no response. If **response** has a value, things went well. In this case, **error** will be undefined, as no error occurred.

```
    const url = `http://api.weatherstack.com/current?access_key=f41e35407998bf659423ffd2ac284503&query=-37.8142,144.9632&   units=m`;

    request({url: url, json: true}, (error, response) => {
        if (error) {
            console.log(`An error occurred: ${error}.`);
        } else {
            const temperature = response.body.current.temperature;
            const precipitation = response.body.current.precip;
            const description = response.body.current.weather_descriptions;
            console.log(`It is currently ${temperature} degrees out. It is ${description} and there is ${precipitation}%    chance of rain.`);
        }
    });
```

I have turned the internet off to test the error handling.

> An error occurred: Error: getaddrinfo ENOTFOUND api.weatherstack.com api.weatherstack.com:80.

This isn't enough error checking. **error** only brings back a low level error message. It will tell us if the internet can't connect but if we provide a faulty latitude or longitude we just get an exception. 

We can check to see if an error was returned in the response.

```
    const url = `http://api.weatherstack.com/current?access_key=f41e35407998bf659423ffd2ac284503&query=-37.8142,144.9632&   units=m`;

    request({url: url, json: true}, (error, response) => {
        if (error) {
            console.log(`An error occurred: ${error}.`);
        } else if (response.body.error) {
            console.log(response.body.error.info);
        } else {
            const temperature = response.body.current.temperature;
            const precipitation = response.body.current.precip;
            const description = response.body.current.weather_descriptions;
            console.log(`It is currently ${temperature} degrees out. It is ${description} and there is ${precipitation}%    chance of rain.`);
        }
    });
```

I remove the latitude and longitude from the API call to generate an error. This is the error message we get back in the response.

> Please specify a valid location identifier using the query parameter.

##### Challenge

Do the same error checking for the Geocoding request.

```
    const locationUrl = 'https://api.mapbox.com/geocoding/v5/mapbox.places/ballarat.json?access_token=pk.   eyJ1IjoiYWxhbnJvYjE3IiwiYSI6ImNrYjM0cWJmaDBoaXUycXFwNWhmc21yZHoifQ.fmtQdmsoTDNr6JWInac8kA';

    request({url: locationUrl, json: true}, (error, response) => {
        if (error) {
            console.log(error);
        } else if (response.body.message) {
            console.log(`Error: ${response.body.message}.`);
        } else {
            const location = response.body.features[0].place_name;
            const long = response.body.features[0].center[0];
            const lat = response.body.features[0].center[1];

            console.log(`The town is ${location}. Its latitude is ${lat} and longitude is ${long}.`);
        }
    });
```

I removed the town in the API request to generate an error.

> Error: Not Found.

### The Callback Function

A callback function is a function that's passed as an argument to another function. Imagine you have FunctionA which gets passed as an argument to FunctionB. FunctionB will do some work and then call FunctionA at some point in the future.

Callback functions are at the core of asynchronous development. When you perform an asynchronous operation, you'll provide Node with a callback function. Node will then call the callback when the async operation is complete. This is how you get access to the results of the async operation, whether it's an HTTP request for JSON data or a query to a database for a user's profile.

```
    setTimeout(() => {
        console.log('The two seconds are up.');
    }, 2000);
```

The code above is an example of a callback function. The ``setTimeout()`` function contains a callback arrow function.

A callback function is nothing more than a function argument that we provide. We know that ``setTimeout()`` will call the function sometime in the future to print out a message.

In this case ``setTimeout()`` is a Node.js function so it is being called asynchronously. This doesn't mean that all callback functions will run asynchronously.

The code below contains a callback function. We have used this pattern before.

```
    const names = ['Alan', 'James', 'Charley'];

    const shortNames = names.filter((name) => {
        return name.length <= 4;
    });

    console.log(shortNames);
```

> [ 'Alan' ]

The code below is a synchronous callback.

```
    const geocode = (address, callback) => {
        const data = {
            latitude: 0,
            longitude: 0
        }

        return data;
    }

    const data = geocode('Melbourne');

    console.log(data);
```

> { latitude: 0, longitude: 0 }

Suppose we were going to use some geocoding method to get the actual coordinates back. We will use ``setTimeout()`` to simulate a HTTP request.

```
    const geocode = (address, callback) => {
        setTimeout(() => {
            const data = {
                latitude: 0,
                longitude: 0
            }

            return data;
        }, 2000);

    }

    const data = geocode('Melbourne');

    console.log(data);
```

This immediately returns before the 2 seconds are up.

> undefined

``geocode()`` is synchronous so runs immediately, returning an undefined because the data didn't have time to return.

``return`` is no longer going to work for us inside a synchronous call. This is where callbacks are going to work for us.

```
    const geocode = (address, callback) => {
        setTimeout(() => {
            const data = {
                latitude: 0,
                longitude: 0
            }

            callback(data);
        }, 2000);
    }

    geocode('Melbourne', (data) => {
        console.log(data);
    });
```

> { latitude: 0, longitude: 0 }

The call to **geocode** provides both arguments, the address and the callback function. Notice that the callback function is expecting a single parameter which it has called **data**. This is where the callback function will get access to the results of the asynchronous operation. You can see where **callback** is called with the data inside the **geocode**
function.

##### Challenge

Try out the callback pattern

1. Define an add function that accepts the correct arguments
2. Use setTimeout to simulate a 2 second delay
3. After 2 seconds are up, call the callback function with the sum
4. Test your work!

```
    add(1, 4, (sum) => {
        console.log(sum) // Should print: 5
    })
```

##### Finished code

```
    const add = (num1, num2, callback) => {
        setTimeout(() => {
            callback(num1 + num2);
        }, 2000);
    }

    add(4, 1, (sum) => {
        console.log(sum);
    });
```

> 5

### Callback Abstraction

Callback functions can be used to abstract complex asynchronous code into a simple reusable function.

Imagine you want to geocode an address from multiple places in your application. You have two options. Option one, you can duplicate the code responsible for making the request. This includes the call to request along with all the code responsible for handling errors. However, this isn't ideal. Duplicating code makes your application unnecessarily
complex and difficult to maintain. The solution is to create a single reusable function that can be called whenever you need to geocode an address.

As an example we are going to refactor the ``app.js`` code so that we have a reusable component that can be called as many times as we want rather than writing the same code with different locations.

In the following example we are going to encode the address in case it contains spaces or other characters. 

```
    const address = 'New York';

    const encodedAdress = encodeURIComponent(address);

    console.log(encodedAdress);
```

Returns.

> New%20York

##### app.js

```
    const geocode = require('./utils/geocode');

    geocode('Ballarat', (error, data) => {
        console.log('Error', error);
        console.log('Data', data);
    });
```

We have abstracted out the ``geocode()`` function and put it in the utils folder. We can now use **geocode.js** as a JavaScript module.

##### geocode.js

```
    const request = require('postman-request');

    const geocode = (address, callback) => {
        const url = 'https://api.mapbox.com/geocoding/v5/mapbox.places/' + encodeURIComponent(address) + '.json?    access_token=pk.eyJ1IjoiYWxhbnJvYjE3IiwiYSI6ImNrYjM0cWJmaDBoaXUycXFwNWhmc21yZHoifQ.fmtQdmsoTDNr6JWInac8kA';

        request({url: url, json: true}, (error, response) => {
            if (error) {
                callback('Unable to connect to location services.');
            } else if (response.body.message) {
                callback('Unable to find location. Try another search.');
            } else {
                callback(undefined, {
                    latitude: response.body.features[0].center[1],
                    longitude: response.body.features[0].center[0],
                    location: response.body.features[0].place_name
                });
            }
        });
    }

    module.exports = geocode;
```

##### Results

> Error undefined
> Data { latitude: -37.5623,
>   longitude: 143.8607,
>   location: 'Ballarat, Victoria, Australia' }


##### Challenge

Create a reusable function for getting the forecast

1. Setup the "forecast" function in utils/forecast.js
2. Require the function in app.js and call it as shown below
3. The forecast function should have three potential calls to callback:
   - Low level error, pass string for error
   - Coordinate error, pass string for error
   - Success, pass forecast string for data (same format as from before)

// forecast(-75.7088, 44.1545, (error, data) => {
//   console.log('Error', error);
//   console.log('Data', data);
// });

##### app.js

```
    const geocode = require('./utils/geocode');
    const forecast = require('./utils/forecast');

    geocode('Ballarat', (error, data) => {
        console.log('Error', error);
        console.log('Data', data);
    });

    forecast(-37.5623, 143.8607, (error, data) => {
      console.log('Error', error);
      console.log('Data', data);
    });
```

##### forecast.js

```
    const request = require('postman-request');

    const forecast = (latitude, longitude, callback) =>  {
        const url = 'http://api.weatherstack.com/current?access_key=f41e35407998bf659423ffd2ac284503&query=' + latitude + ','   + longitude + '&units=m';

        request({url: url, json: true}, (error, response) => {
            if (error) {
                callback('Unable to connect to weather service.');
            } else if (response.body.error) {
                callback('Unable to find location.');
            } else {
                const temperature = response.body.current.temperature;
                const precipitation = response.body.current.precip;
                const description = response.body.current.weather_descriptions;
                callback(undefined, `It is currently ${temperature} degrees out. It is ${description} and there is $    {precipitation}% chance of rain.`);
            }
        });
    }

    module.exports = forecast;
```

##### Results for forecast

> Error undefined
> Data It is currently 9 degrees out. It is Partly cloudy and there is 0% chance of rain.

### Callback Chaining

We'll learn how to run one asynchronous operation only after another asynchronous operation is complete. That'll allow you to use the output from geocoding as the input for fetching the weather.

When working with async code, you'll often find out that you need to use the results from one async operation as the input for another async operation. This is something we need to do in the weather application too. Step one is to geocode the address. Step two is to use the coordinates to fetch the weather forecast. You can't start step two until step one is
complete. 

You can start one operation after another finishes by using callback chaining. You can see an example of this in the code below.

```
    const geocode = require('./utils/geocode');
    const forecast = require('./utils/forecast');

    geocode('Cairns', (error, data) => {
        console.log('Error', error);
        console.log('Data', data);
        forecast(data.latitude, data.longitude, (error, data) => {
          console.log('Error', error);
          console.log('Data', data);
        });
    });
```

> Error undefined
> Data { latitude: -16.9205,
>   longitude: 145.7719,
>   location: 'Cairns, Queensland, Australia' }
> Error undefined
> Data It is currently 24 degrees out. It is Partly cloudy and there is 0% chance of rain.

What happens if ``geocode()`` fails first? We need to add error handling so ``forecast()`` doesn't run.

```
    const geocode = require('./utils/geocode');
    const forecast = require('./utils/forecast');

    geocode('Ballarat', (error, data) => {
      if (error) {
        return console.log('Error', error);
      }

      forecast(data.latitude, data.longitude, (error, forecastData) => {
        if (error) {
          return console.log('Error', error);
        }

        console.log(data.location);
        console.log(forecastData);
      });
    });
```

> Ballarat, Victoria, Australia
> It is currently 5 degrees out. It is Partly cloudy and there is 0% chance of rain.

There were two options for error handling. The first is to use ``if/else`` statements but it is preferable to use an ``if`` statement to **return** the error immediately.

##### Challenge

Accept location via a command line argument.

1. Access the command line argument without yargs.
2. Use the string value as the input for geocode.
3. Only geocode if a command line was provided.
4. Test your work with a couple of locations.

```
    const geocode = require('./utils/geocode');
    const forecast = require('./utils/forecast');

    // test for input
    if (process.argv.length < 3) {
      return console.log('Please enter a location!');
    }

    const location = process.argv[2];

    geocode(location, (error, data) => {
      if (error) {
        return console.log('Error', error);
      }

      forecast(data.latitude, data.longitude, (error, forecastData) => {
        if (error) {
          return console.log('Error', error);
        }

        console.log(data.location);
        console.log(forecastData);
      });
    });
```

##### test 1

node app "Sydney, NSW"

> Sydney, New South Wales, Australia
> It is currently 15 degrees out. It is Partly cloudy and there is 0.1% chance of rain.

##### test 2

node app "London, UK"

> London, Greater London, England, United Kingdom
> It is currently 12 degrees out. It is Overcast and there is 0.1% chance of rain.

### ES6 Aside: Object Property Shorthand and Destructuring

ES6 has done wonders making JavaScript easier to use. We will explore a couple of features that make it easier to work with objects.

#### Property Shorthand

The property shorthand makes it easier to define properties when creating a new object. It provides a shortcut for defining a property whose value comes from a variable of the same name. You can see this in the example below where a user object is created. The name property gets its value from a variable also called name.

```
    const name = 'Alan';
    const userAge = 68;

    const user = {
        name: name,
        age: userAge,
        location: 'Ballarat'
    }

    console.log(user);
```

> { name: 'Alan', age: 68, location: 'Ballarat' }

We can use the ES6 object property shorthand.

On the first property ``name`` we are using the ES6 object property shorthand.

**Note:** The variable names have to match the names in the object. This is why the object property shorthand doesn't work for the ``age`` property.

```
    // Object property shorthand
    const name = 'Alan';
    const userAge = 68;

    const user = {
        name,
        age: userAge,
        location: 'Ballarat'
    }

    console.log(user);
```

> { name: 'Alan', age: 68, location: 'Ballarat' }

#### Object Destructuring

The second ES6 feature is object destructuring. Object destructuring gives you a syntax for pulling properties off of objects and into standalone variables. This is useful when working with the same object properties throughout your code. Instead of writing ``user.name`` a dozen times, you could destructure the property into a name variable.

```
    // Object destructuring
    const product = {
        label: 'Red notebook',
        price: 5,
        stock: 201,
        salePrice: undefined
    }

    // this is long winded.
    // const label = product.label;
    // const stock = product.stock;

    const {label, price, stock, salePrice} = product;

    console.log(label);
    console.log(price);
    console.log(stock);
    console.log(salePrice);
```

> Red notebook
> 5
> 201
> undefined

We can rename the variables.

```
// Object destructuring
const product = {
    label: 'Red notebook',
    price: 5,
    stock: 201,
    salePrice: undefined
}

const {label:productLabel, price, stock:stockLevel, salePrice} = product;

console.log(productLabel);
console.log(price);
console.log(stockLevel);
console.log(salePrice);
```

Produces the same result as above.

We can also add a default value into a property if a value is undefined.

```
    // Object destructuring
    const product = {
        label: 'Red notebook',
        price: 5,
        stock: 201,
        salePrice: undefined
    }

    const {label:productLabel, price, stock:stockLevel, salePrice = 5} = product;

    console.log(productLabel);
    console.log(price);
    console.log(stockLevel);
    console.log(salePrice);
```

> Red notebook
> 5
> 201
> 5

**Note:** It is important to note that the default value will only be used if a property is undefined, otherwise it will use the value entered.

We can use a shorthand for destructuring.

```
    // Object destructuring
    const product = {
        label: 'Red notebook',
        price: 5,
        stock: 201,
        salePrice: undefined
    }

    const {label:productLabel, price, stock:stockLevel, salePrice = 5} = product;

    const transaction = (type, { }) => {
        console.log(type);
        console.log(productLabel);
        console.log(price);
        console.log(stockLevel);
        console.log(salePrice);
    }

    transaction('order', product);
```

or we could just destructure a couple of properties.

```
    const product = {
        label: 'Red notebook',
        price: 5,
        stock: 201,
        salePrice: undefined
    }

    const transaction = (type, { label, stock }) => {
        console.log(type, label, stock);
    }

    transaction('order', product);
```

> order Red notebook 201

##### Challenge

Use destructuring and object property shorthand in the weather app.

1. Use destructuring in app.js, forecast.js and geocode.js.
2. Use property shorthand in forecast.js and geocode.js.
3. Test your work and make sure the app still works.

##### app.js

```
    const geocode = require('./utils/geocode');
    const forecast = require('./utils/forecast');

    // test for input
    if (process.argv.length < 3) {
      return console.log('Please enter a location!');
    }

    const location = process.argv[2];

    geocode(location, (error, { latitude, longitude, location } = {}) => {
      if (error) {
        return console.log('Error', error);
      }

      forecast(latitude, longitude, (error, forecastData) => {
        if (error) {
          return console.log('Error', error);
        }

        console.log(location);
        console.log(forecastData);
      });
    });
```

In the ``geocode()`` call we have destructured the **data** object to,

```
    { latitude, longitude, location }
```

This saves us having to call each property by *data.latitude*, etc.

##### geocode.js

```
    const request = require('postman-request');

    const geocode = (address, callback) => {
        const url = 'https://api.mapbox.com/geocoding/v5/mapbox.places/' + encodeURIComponent(address) + '.json?    access_token=pk.eyJ1IjoiYWxhbnJvYjE3IiwiYSI6ImNrYjRtMHR3bzBrYTIycm85bm9zNnhwb3oifQ.hVxBRq8r_RSiT8edHFeyzw';

        request({url, json: true}, (error, { body }) => {
            if (error) {
                callback('Unable to connect to location services.');
            } else if (body.message) {
                callback('Unable to find location. Try another search.');
            } else {
                callback(undefined, {
                    latitude: body.features[0].center[1],
                    longitude: body.features[0].center[0],
                    location: body.features[0].place_name
                });
            }
        });
    }

    module.exports = geocode;
```

The places we use the property object is in the **request** call.

```
    request({url: url, json: true}, (error, response) => {
```

change to,

```
    request({url, json: true}, (error, { body }) => {
```

we simplify **url** property and change **response** to call the only property we require, **body**. Now instead of calling properties with **response.body** we only have to use **body**.

We do the same changes to forecast.js.

##### forecast.js

```
    const request = require('postman-request');

    const forecast = (latitude, longitude, callback) =>  {
        const url = 'http://api.weatherstack.com/current?access_key=f41e35407998bf659423ffd2ac284503&query=' + latitude + ','   + longitude + '&units=m';

        request({ url, json: true }, (error, { body }) => {
            if (error) {
                callback('Unable to connect to weather service.');
            } else if (body.error) {
                callback('Unable to find location.');
            } else {
                const tempObject = {
                    temperature: body.current.temperature,
                    precipitation: body.current.precip,
                    description: body.current.weather_descriptions
                }

                const { temperature, precipitation, description } = tempObject;

                callback(undefined, `It is currently ${temperature} degrees out. It is ${description} and there is $    {precipitation}% chance of rain.`);
            }
        });
    }

    module.exports = forecast;
```

### HTTP Requests Without a Library

While the request library is great, it's not necessary if you want to make HTTP requests from Node. We will now make an HTTP request without request.

#### The HTTP Module

Node.js provides two core modules for making HTTP requests. The http module can be used to make http requests and the https module can be used to make https requests. One great feature about **request** is that it provides a single module that can make both http and https requests.

The code below uses the http module to fetch the forecast from the Wetherstack. Notice there's a lot more required to get things working. Separate callbacks are required for incoming chunks of data, the end of the response, and the error for the request. This means you'll likely recreate your own function similar to **request** to make your life easier. It's best to stick with a tested and popular library like request.

```
    const http = require('http');

    const url = 'http://api.weatherstack.com/current?access_key=f41e35407998bf659423ffd2ac284503&query=56.01681 -3.60891&   units=m';;


    const request = http.request(url, (response) => {
        let data = '';

        response.on('data', (chunk) => {
            data = data + chunk.toString();
        });

        response.on('end', () => {
            const body = JSON.parse(data);
            console.log(body);
        });
    });

    request.on('error', (error) => {
        console.log('An error', error)
    });

    request.end();
```

> { request:
>    { type: 'LatLon',
>      query: 'Lat 56.02 and Lon -3.61',
>      language: 'en',
>      unit: 'm' },
>   location:
>    { name: 'Bo\'ness',
>      country: 'United Kingdom',
>      region: 'Falkirk',
>      lat: '56.016',
>      lon: '-3.614',
>      timezone_id: 'Europe/London',
>      localtime: '2020-06-08 08:20',
>      localtime_epoch: 1591604400,
>      utc_offset: '1.0' },
>   current:
>    { observation_time: '07:20 AM',
>      temperature: 9,
>      weather_code: 122,
>      weather_icons:
>       [ 'https://assets.weatherstack.com/images/wsymbols01_png_64/wsymbol_0004_black_low_cloud.png' ],
>      weather_descriptions: [ 'Overcast' ],
>      wind_speed: 9,
>      wind_degree: 80,
>      wind_dir: 'E',
>      pressure: 1020,
>      precip: 0,
>      humidity: 66,
>      cloudcover: 100,
>      feelslike: 8,
>      uv_index: 5,
>      visibility: 10,
>      is_day: 'yes' } }
