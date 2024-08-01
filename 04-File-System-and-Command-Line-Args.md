# File System and Command Line Arguments

## Getting Input from Users

I can't think of a single useful application that doesn't get input from the users. Whether it's
their email, location, or age, getting input is essential for creating real-world apps. We need to learn how to set up command line arguments that allow users to pass data into our application.

#### Accessing Command Line Arguments

Command line arguments are values passed into your application from the terminal. Your Node.js application can access the command line arguments that were provided using ``process.argv``. This array contains at least two items. The first is the path to the Node.js executable. The second is the path to the JavaScript file that was executed. Everything after that is a command line argument.

Run the command:

```
    node app Alan
```

**node app** is used to run the app.js script file. *Alan* is the argument. We can access the command line arguments with:

```
    console.log(process.argv);
```

Which prints out.

> [ 'C:\\Program Files\\nodejs\\node.exe',
>  'D:\\WebDev\\Node\\node-course\\notes-app\\app',
>  'Alan' ]

The first argument is the program we are running. The second argument is the script file we are running. The third argument is the text we typed in, *Alan*.

We can grab an argument by its index value. In our case we want to get the text entered.

```
    console.log(process.argv[2]);
```

> Alan

We can use this to run a command. For example if we want to add a new note.

```
    node app add
```

Our code is:

```
    const command = process.argv[2];

    if (command === 'add') {
        console.log('Add a new note...');
    } else if (command === 'remove') {
        console.log('Removing note...');
    }
```

> $ node app add
> Add a new note...

Run again.

> $ node app remove
> Removing note...

## Argument Parsing with Yargs - Part 1

Node.js provides a bare-bones way to access command line arguments. While it's a good start, it doesn't provide any way to parse more complex command line arguments. We are going to use Yargs to easily set up a more complex set of arguments for
your application.

#### Setting Up Yargs

First, install **Yargs** in your project.

```
    npm i yargs
```

Now, yargs can be used to make it easier to work with command line arguments. The following code will show the difference between standard argument parsing and yargs parsing.

```
    const chalk = require('chalk');
    const yargs = require('yargs');
    const getNotes = require('./notes.js');

    console.log(process.argv);
  
    console.log();

    console.log(yargs.argv);
```

> [ 'C:\\Program Files\\nodejs\\node.exe',
>   'D:\\WebDev\\Node\\node-course\\notes-app\\app',
>   'add',
>   '--title=This is my new title' ]
> 
> { _: [ 'add' ], title: 'This is my new title', '$0': 'app' }

Now, run the following command.

```
    node app --help
```

> Options:
>   --help     Show help                     [boolean]
>   --version  Show version number           [boolean]

We can check the version:

```
    node app --version
```

> 1.0.0

We can modify the version of yargs that we are using by:

```
    yargs.version('1.1.0');
```

Run this:

```
    node app --version
```

> 1.1.0

Add the following code:

```
    yargs.command({
        command: 'add',
        describe: 'Add a new note.',
        handler: function () {
            console.log('Adding a new note...');
        }
    });

    console.log(yargs.argv);
```

Now we can run the help again.

```
    node app --help
```

> Commands:
>   app add  Add a new note.
>
> Options:
>   --help     Show help                   [boolean]
>   --version  Show version number         [boolean]

Now, we will go ahead and use the command.

```
    node app add
```

Returns.

> Adding a new note...
> { _: [ 'add' ], '$0': 'app' }

Now, let's add a new command

```
    // Create remove command
    yargs.command({
        command: 'remove',
        describe: 'Remove the current note.',
        handler: function () {
            console.log('Removing a note...');
        }
    });

    console.log(yargs.argv);
```

Run.

```
    node app remove
```

> Removing a note...
> { _: [ 'remove' ], '$0': 'app' }

##### Challenge

Add two new commands

1. Setup command to support "list" and print placeholder message for now.
2. Setup command to support "read" and print placeholder message for now.
3. Test your work by running both commands to ensure correct output.

Code.

```
    // Create list command
    yargs.command({
        command: 'list',
        describe: 'List all notes.',
        handler: function () {
            console.log('Listing the notes...');
        }
    });

    // Create read command
    yargs.command({
        command: 'read',
        describe: 'read current note.',
        handler: function () {
            console.log('Read the current note...');
        }
    });

    console.log(yargs.argv);
```

Run.

```
    node app list
```

> Listing the notes...
> { _: [ 'list' ], '$0': 'app' }

Run.

```
    node app read
```

> Read the current note...
> { _: [ 'read' ], '$0': 'app' }

Now run the help command.

```
    node app --help
```

> app [command]
>
> Commands:
>   app add     Add a new note.
>   app remove  Remove the current note.
>   app list    List all notes.
>   app read    read current note.

## Argument Parsing with Yargs - Part II

We will continue to explore Yargs. The goal is to allow users to pass in the title and body of their notes using command line options. This same technique could be used to allow users to pass in data such as their name, email, or address.

We are going to add a title to the **add** command.

```
    // Create add command
    yargs.command({
        command: 'add',
        describe: 'Add a new note.',
        builder: {
            title: {
                describe: 'Note title.'
            }
        },
        handler: function (argv) {
            console.log('Adding a new note...', argv);
        }
    });
```

Run.

```
    node app add --title="Shopping list."
```

> Adding a new note... { _: [ 'add' ], title: 'Shopping list.', '$0': 'app' }
> { _: [ 'add' ], title: 'Shopping list.', '$0': 'app' }

we also have the option to force the user to add a title when adding a new note.

```
    // Create add command
    yargs.command({
        command: 'add',
        describe: 'Add a new note.',
        builder: {
            title: {
                describe: 'Note title.',
                demandOption: true
            }
        },
        handler: function (argv) {
            console.log('Adding a new note...', argv);
        }
    });
```

Run.

```
    node app add
```

> app add
>
> Add a new note.
>
> Options:
>   --help     Show help                  [boolean]
>   --version  Show version number        [boolean]
>   --title    Note title.               [required]
>
> Missing required argument: title

We also want to ensure that the title is always a string.

```
    // Create add command
    yargs.command({
        command: 'add',
        describe: 'Add a new note.',
        builder: {
            title: {
                describe: 'Note title.',
                demandOption: true,
                type: 'string'
            }
        },
        handler: function (argv) {
            console.log('Adding a new note...', argv);
        }
    });
```

Run.

```
    node app add -- title=
```

> app add
>
> Add a new note.
>
> Options:
>   --help     Show help                         [boolean]
>   --version  Show version number               [boolean]
>   --title    Note title.             [string] [required]
>
> Missing required argument: title

In the code we are using the following line to print the arguments and this is important.

```
    console.log(yargs.argv);
```

If we don't add this the code won't print out the arguments. There is another way to do this.

```
    yargs.parse();
```

Will print out the same message as above.

Now we want to be able to grab the *title* value and we can do this by:

```
    // Create add command
    yargs.command({
        command: 'add',
        describe: 'Add a new note.',
        builder: {
            title: {
                describe: 'Note title.',
                demandOption: true,
                type: 'string'
            }
        },
        handler: function (argv) {
            console.log('Title: ' + argv.title);
        }
    });
```

Run.

```
    node app add --title="Shopping list."
```

> Title: Shopping list.

##### Challenge

Add an option to yargs.

1. Setup a body option for the add command.
2. Configure a description, make it required and make it a string type.
3. Log the body value in the handler function.
4. Test your work.

Code.

```
    // Create add command
    yargs.command({
        command: 'add',
        describe: 'Add a new note.',
        builder: {
            title: {
                describe: 'Note title.',
                demandOption: true,
                type: 'string'
            },
            body: {
                describe: 'Note body.',
                demandOption: true,
                type: 'string'
            }
        },
        handler: function (argv) {
            console.log('Title: ' + argv.title);
            console.log('Body: ' + argv.body);
        }
    });
```

Run.

```
    node app add --title="Shopping list." --body="This is my list of shopping items."
```

> Title: Shopping list.
> Body: This is my list of shopping items.

Run.

```
    node app add --title="Shopping list."
```

> app add
>
> Add a new note.
>
> Options:
>   --help     Show help                                                 [boolean]
>   --version  Show version number                                       [boolean]
>   --title    Note title.                                     [string] [required]
>   --body     Note body.                                      [string] [required]
>
> Missing required argument: body

## Storing Data with JSON

We need to learn how to work with JSON. JSON, which stands for JavaScript Object Notation, is a lightweight data format. JSON makes it easy to store or transfer data. 

You'll be using it in this application to store users notes in the file system.

#### Working with JSON

Since JSON is nothing more than a string, it can be used to store data in a text file or transfer data via HTTP requests between two machines.

JavaScript provides two methods for working with JSON. The first is ``JSON.stringify`` and the second is ``JSON.parse``. ``JSON.stringify`` converts a JavaScript object into a JSON string, while ``JSON.parse`` converts a JSON string into a JavaScript object.

An example of working with JSON.

```
    const book = {
        title: 'The Rich Man\'s House',
        author: 'Andrew McGahan'
    }

    const bookJSON = JSON.stringify(book);
    console.log(bookJSON);
```

> {"title":"The Rich Man's House","author":"Andrew McGahan"}

The opposite of ``JSON.stringify`` is ``JSON.parse``. ``JSON.parse`` takes in the JSON string and passes back a JSON object.

```
    const newBook = JSON.parse(bookJSON);

    console.log('Book: %s -- %s', newBook.author, newBook.title);
```

> Book: Andrew McGahan -- The Rich Man's House

Next we are going to save our JSON text to a file.

```
    const fs = require('fs');

    const book = {
        title: 'The Rich Man\'s House',
        author: 'Andrew McGahan'
    }

    const bookJSON = JSON.stringify(book);

    fs.writeFileSync('books.json', bookJSON);
```

We can view the file with the command.

```
    cat books.json
```

> {"title":"The Rich Man's House","author":"Andrew McGahan"}

Now we want to comment out all of the writing code and create some code to read in the JSON file.

```
    const fs = require('fs');

    const dataBuffer = fs.readFileSync('books.json');

    console.log(dataBuffer);
```

At this stage the data is actually binary data and we can see this if we dump ``dataBuffer`` to the console.

> <Buffer 7b 22 74 69 74 6c 65 22 3a 22 54 68 65 20 52 69 63 68 20 4d 61 6e 27 73 20 48 6f 75 73 65 22 2c 22 61 75 74 68 6f 72 22 3a 22 41 6e 64 72 65 77 20 4d ... >

To make the data more user friendly we can use JSON.parse() to format it back into an object.

```
    const fs = require('fs');

    const dataBuffer = fs.readFileSync('books.json');

    const book = JSON.parse(dataBuffer);

    console.log('Book: %s -- %s', book.author, book.title);
```

> Book: Andrew McGahan -- The Rich Man's House

I could have also used this code to change the binary data into a string.

```
    console.log(dataBuffer.toString());
```

> {"title":"The Rich Man's House","author":"Andrew McGahan"}

##### Challenge

Work with JSON and the file system

1. Load and parse the JSON data.
2. Change the name and age properties to your data.
3. Stringify the changed object and overwrite the original data.
4. Test your data by viewing the data in the JSON file.

Code.

```
    const fs = require('fs');

    const dataBuffer = fs.readFileSync('data.json');

    const person = JSON.parse(dataBuffer);

    person.name = 'Alan';
    person.planet = 'Mars';
    person.age = 68;

    const personJSON = JSON.stringify(person);

    fs.writeFileSync('data.json', personJSON);
```

Run.

```
    cat data.json
```

> {"name":"Alan","planet":"Mars","age":68}

## Adding a note

##### app.js

```
    const chalk = require('chalk');
    const yargs = require('yargs');
    const notes = require('./notes.js');

    yargs.version('1.1.0');

    // Create add command
    yargs.command({
        command: 'add',
        describe: 'Add a new note.',
        builder: {
            title: {
                describe: 'Note title.',
                demandOption: true,
                type: 'string'
            },
            body: {
                describe: 'Note body.',
                demandOption: true,
                type: 'string'
            }
        },
        handler: function (argv) {
            notes.addNote(argv.title, argv.body);
        }
    });
```

We have to change the ``require`` call for *notes.js* as we need to access multiple functions. We store the different functions in an object named **notes**. To use one of the functions from *notes.js* we use dot notation.

```
    notes.addNote(argv.title, argv.body);
```

In the case above we are calling ``addNote()`` from *notes.js*.

##### notes.js

```
    const fs = require('fs');

    const getNotes = function () {
        return 'Your notes...';
    }

    const addNote = function (title, body) {
        const notes = loadNotes();
        const duplicateNotes = notes.filter(function (note) {
            return note.title.toLowerCase() === title.toLowerCase();
        });

        if (duplicateNotes.length === 0) {
            notes.push({
                title: title,
                body: body
            });

            saveNotes(notes);
            console.log('New note added.');
        } else {
            console.log('Duplicate note, not added.');
        }
    }

    const saveNotes = function (notes) {
        const dataJSON = JSON.stringify(notes);
        fs.writeFileSync('notes.json', dataJSON);
    }

    const loadNotes = function () {
        try {
            const dataBuffer = fs.readFileSync('notes.json');
            const dataJSON = dataBuffer.toString();

            return JSON.parse(dataJSON);
        } catch (e) {
            return [];
        }
    }

    module.exports = {
        getNotes: getNotes,
        addNote: addNote
    }
```

The first thing to note is the ``module.exports`` object at the bottom of the code.

```
    module.exports = {
        getNotes: getNotes,
        addNote: addNote
    }
```

This is how we export multiple functions from this file to be used in other JavaScript files.

In the ``addNote()`` function we are checking the title field for duplicate titles. If one is found we are going to ignore it. We filter through the existing notes to see if the title already exists in the notes.json.

## Removing a note

##### Challenge

Setup command option and function

1. Setup the "Remove" command to take a required "--title" option.
2. Create and export a removeNote function from notes.js.
3. Call removeNote in remove command handler.
4. Have removeNote log the title of the note to be removed.
5. Test your work using: node app remove --title "Some title to be removed".

##### app.js

```
    // Create remove command
    yargs.command({
        command: 'remove',
        describe: 'Remove the current note.',
        handler: function (argv) {
            notes.removeNote(argv.title);
        }
    });
```

##### notes.js

```
    const removeNote = function (title) {
        const notes = loadNotes();

        const notesToKeep = notes.filter(function (note) {
            return note.title.toLowerCase() !== title.toLowerCase();
        });

        if (notes.length > notesToKeep.length) {
            console.log(chalk.green.inverse('Note removed!'))
            saveNotes(notesToKeep);
        } else {
            console.log(chalk.red.inverse('No note found!'));
        }
    }

    module.exports = {
        getNotes: getNotes,
        addNote: addNote,
        removeNote: removeNote
    }
```

In *notes.js* we are going to select all of the notes that don't match the title we want to remove. Once we get these we can save the notes and the note that we wanted to remove will be left out of *notes.json*.

## More notes on arrow functions

We are going to create an object with a standard function.

```
    const event = {
        name: 'Birthday Party',
        printGuestList: function () {
            console.log('Guest list for ' + this.name);
        }
    }

    event.printGuestList();
```

> Guest list for Birthday Party

Now, we are going to change it into an arrow function.

```
const event = {
    name: 'Birthday Party',
    printGuestList: () => {
        console.log('Guest list for ' + this.name);
    }
}

event.printGuestList();
```

> Guest list for undefined

This doesn't work as anticipated. Clearly it is unable to find the *name* variable. This is because arrow functions don't bind their own ``this`` value. That means that we don't have access to ``this`` in reference to this object.

Arrow functions aren't suited to methods in objects.

There is actually an ES6 method shorthand that allows us to have a shorter syntax while still having access to standard function features like the  ``this`` binding.

```
    const event = {
        name: 'Birthday Party',
        printGuestList() {
            console.log('Guest list for ' + this.name);
        }
    }

    event.printGuestList();
```

> Guest list for Birthday Party

The code above isn't an arrow function. It is just using an alternative syntax available to use when we are setting up methods on objects.

We are going to add on to the previous example.

```
    const event = {
        name: 'Birthday Party',
        guestList: ['Alan', 'Justin', 'Jen'],
        printGuestList() {
            console.log('Guest list for ' + this.name);

            this.guestList.forEach(function (guest) {
                console.log(guest + ' will be attending the ' + this.name);
            });
        }
    }

    event.printGuestList();
```

> Guest list for Birthday Party
> Alan will be attending the undefined
> Justin will be attending the undefined
> Jen will be attending the undefined

Once again this code doesn't work. This is because the new standard function we created has its own ``this`` binding and in this case we don't want it to have a ``this`` binding. We want it to access the ``this`` binding of its parent function (printGuestList()) because there I was able to access the ``this`` binding for ``this.name``.

Arrow functions can solve the problem we have.

```
    const event = {
        name: 'Birthday Party',
        guestList: ['Alan', 'Justin', 'Jen'],
        printGuestList() {
            console.log('Guest list for ' + this.name);

            this.guestList.forEach((guest) => {
                console.log(guest + ' will be attending the ' + this.name);
            });
        }
    }

    event.printGuestList();
```

> Guest list for Birthday Party
> Alan will be attending the Birthday Party
> Justin will be attending the Birthday Party
> Jen will be attending the Birthday Party

Swapping out the standard function for an arrow function works. Arrow functions don't bind their own ``this`` value so that means we have access to the ``this`` value outside of our function.

##### Challenge

Create method to get incomplete tasks

1. Define getTasksToDo method
2. Use filter to to return just the uncompleted tasks (arrow function)
3. Test your work by running the script

```
    const tasks = {
        tasks: [{
            text: 'Grocery shopping',
            completed: true
        },{
            text: 'Clean yard',
            completed: false
        }, {
            text: 'Film course',
            completed: false
        }]
    }

    console.log(tasks.getTasksToDo());
```

##### Completed code

```
    const tasks = {
        tasks: [{
            text: 'Grocery shopping',
            completed: true
        },{
            text: 'Clean yard',
            completed: false
        }, {
            text: 'Film course',
            completed: false
        }],
        getTasksToDo() {
            const tasksToDo = this.tasks.filter((task) => {
                return task.completed === false;
            });

            return tasksToDo;
        }
    }

    console.log(tasks.getTasksToDo());
```

> [ { text: 'Clean yard', completed: false },
>   { text: 'Film course', completed: false } ]

We can refactor this to a shorthand method call.

```
    getTasksToDo() {
        return this.tasks.filter((task) => task.completed === false);
    }
```

##### Challenge

Refacter all functions in notes-app

1. If a function is a method, use ES6 method function syntax.
2. Otherwise use the most concise arrow function possible.
3. Test your work.

###### notes.js

```
    const chalk = require('chalk');
    const fs = require('fs');

    const getNotes = () => 'Your notes...';

    const addNote = (title, body) => {
        const notes = loadNotes();
        const duplicateNotes = notes.filter((note) => note.title.toLowerCase() === title.toLowerCase());

        if (duplicateNotes.length === 0) {
            notes.push({
                title: title,
                body: body
            });

            saveNotes(notes);
            console.log('New note added.');
        } else {
            console.log('Duplicate note, not added.');
        }
    }

    const saveNotes = (notes) => {
        const dataJSON = JSON.stringify(notes);
        fs.writeFileSync('notes.json', dataJSON);
    }

    const loadNotes = () => {
        try {
            const dataBuffer = fs.readFileSync('notes.json');
            const dataJSON = dataBuffer.toString();

            return JSON.parse(dataJSON);
        } catch (e) {
            return [];
        }
    }

    const removeNote = (title) => {
        const notes = loadNotes();

        const notesToKeep = notes.filter((note) => note.title.toLowerCase() !== title.toLowerCase());

        if (notes.length > notesToKeep.length) {
            console.log(chalk.green.inverse('Note removed!'))
            saveNotes(notesToKeep);
        } else {
            console.log(chalk.red.inverse('No note found!'));
        }
    }

    module.exports = {
        getNotes: getNotes,
        addNote: addNote,
        removeNote: removeNote
    }
```

##### Challenge

1. Create and export listNotes from notes.js. Header - "Your notes" from chalk. Print note title from each note.
2. Call listNotes from command handler.
3. Test your work.

```
    const loadNotes = () => {
        try {
            const dataBuffer = fs.readFileSync('notes.json');
            const dataJSON = dataBuffer.toString();

            return JSON.parse(dataJSON);
        } catch (e) {
            return [];
        }
    }
```

##### Challenge

Wire up the read command

1. Setup --title option for read command
2. Create readNote in notes.js. Search for note by title. Find note and print note title (styled) and body (unstyled). No note found, print error in red.
3. Have the command handler call the function.
4. Test your work by running a couple of commands.

```
    const readNote = (title) => {
        const notes = loadNotes();

        const foundNote = notes.find((note) => note.title.toLowerCase() === title.toLowerCase());

        if (foundNote) {
            console.log(chalk.green(foundNote.title) + ' \n\t' + foundNote.body);
        } else {
            console.log(chalk.red.inverse('Note not found!'));
        }
    }
```

###### Results

$ node app read --title="Take out the rubbish"

> Take out the rubbish
>         Every Wednesday night.

$ node app read --title="dog"

> Note not found!
