# Goal

Frameworks are powerful and useful tools. They abstract away a lot of the painful details that go into creating a web application. Theoretically, this means that new developers can be helpful to development quicker, and older developers can be even more efficient, as they don't have to dredge through tedious, lower-level details.

However, a framework's greatest strength is also its greatest weakness. Newer developers that only learn how to use a framework, and never learn what the framework is abstracting away, may be less efficient in the long run. They may not be entirely aware of what they are even doing when using that framework. And, in the event the framework fails, that developer may not know how to fix it, because they are unaware of what the framework is doing in the first place.

Thus, I hope to fight against that problem by writing this. By using an example project, I want to build a web app from the ground up, and slowly upgrade it into a more modern-looking app that uses frameworks. Doing this, we should be able to see why certain design choices and frameworks are so ubiquitious, and should gain a greater appreciation for the frameworks that we rely on.

# What is a web app?

When doing anything big or complicated it's important to ask a few questions. The first two are *why* and *what*. If we know the answers to these questions, our goals and the required steps to reach them should become a lot more obvious. Once we have concrete goals in mind, it's easy to reach your greater goal step-by-step. We've already covered *why* we're building a web app. I just stated that in the goals section, and I'm sure you can come up with many other reasons why you might build one. Generally, it's to solve some problem.

The important question here is *what* a web app is. Most of us have a pretty intuitive understanding of one. It is a website. Something like Google, Youtube or Netflix. You enter a URL into a browser, it takes you to a page, and you can click buttons and do things. However, we're hoping for a defintion that helps us understand the technical side of a website, and that one is too vague to tell us anything more about web development. So, lets dive deeper.

## A Basic Web App Definition

A web app is entered by entering a URL into a browser. The computer running the browser is known as the *client*. This URL connects you to another computer called a *server*. This server serves content to the client. The client then uses the content given by the server to input data. This data is sent to the server, which causes it to serve new content to the client. The client sending data to the server is called a *request*, and the server sending data back to the client is called a *response*. This is shown in the diagram below.

![How Web Apps Work  Web Application Architecture Simplified  Reinvently](https://reinvently.com/wp-content/uploads/2019/08/scheme.jpg)

The web app that we write is running on the server. So, a web app's job is to take incoming requests, process them, and send back adequate responses containing content for the client to use. This "content" is, of course, our frontend. The HTML, CSS and Javascript that our users see and use in browser to send requests. Thus, when we say that we are going to "write a web app in Rust", we are really saying that we are going to write a program in Rust that processes incoming requests, and sends back appropriate responses in the form of HTML, CSS and Javascript.

## A More Modern Web App Definition

This definition has been shaken up a little bit in recent years. Frameworks like React change how a web app is structured. With a React app created in `create-react-app` you have a structure more like the following. Instead of the user connecting directly to the server, they instead connect to a computer that serves back a compiled frontend made using React. This computer, when given a request by the user, sends it to another computer that contains the server. The server sends the data back to the computer with the React code, and the React code then uses that data to send a response to the user.

In this architecture, there are two computers that the user is interfacing with. One holds the React frontend, and the other holds some backend that only communicates data. The computer with the frontend uses the data from the backend to create a response to send to the user.

This architecture comes with some advantages for developers, as writing the frontend and backend can become a lot easier. Sometimes the backend basically just saves and returns data that the frontend tells it to, and does almost no processing of the data. This is why some people say that modern websites are just fancy wrappers for databases. The backend can basically just be a database and the frontend is just a really complicated wrapper around it.

Note that this definition is not particularly useful to the web app we will be writing, but since so many web apps end up being structured like this, I thought it worth mentioning.

# A Note About Requests and Responses

Sadly, there is still a little bit more technical jargon to understand before we can dive into the meat of our example. As I laid out in the previous section, users send requests to the server and the server serves back responses. However, it's important to understand how those requests and responses look a little bit.

## HTTP Protocol

The server and client communicate via HTTP. It is a protocol, which means it is a set of rules that determine how data is transmitted. In more familiar terms, it is a way that server and clients have agreed to communicate with each other.

Requests in HTTP look like the following

![A basic HTTP request](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview/http_request.png)

As you can see, we send a method, a certain URL (that's what "Path" is), the version of the protocol, some headers that convey information to a sever, and a body (depending on the method. A `GET` method does not have a body, which is why none is shown here). Based on the information given here, a server determines a proper response, which looks like this.

![HTTP Response image](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview/http_response.png)

Instead of a method and path, we have a status code and message, which tells us what happened with the request. We still have the version of the protocol, we still have headers and we still have a body. Most of the time, in a web app structured like ours, the body of the response is HTML, and the browser will display that HTML.

This is how most, if not all, browsers send and expect to recieve data, and, thus, our web app will want to accomodate this.

## Query Language

In recent years, a query language called [GraphQL](https://graphql.org/) has become quite popular. If you are someone who's aware of the above section, you may be confused as to how something like GraphQL works, because you send out requests and responses that look like this

```graphql
{
  me {
    name
  }
}
```

and, somehow, the browser and server are still able to understand. How does this work? Well, query languages are basically fancy ways to make HTTP requests and responses. You write out a certain piece of code in something like GraphQL, and some program converts that into a series of HTTP requests and responses.

You may end up using or seeing query languages if you continue to make web apps, and the important thing to remember is they are still using the same exact tech under the hood, but, just like frameworks, it is a layer of abstraction to help make web development easier.

# Final Note on the Jargon

Finally, before we move on, I just want to recommend this article from MDN Web Docs: [An overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview). It goes over all the stuff I just covered: servers, clients, requests, responses and HTTP. If you want to hear all this explained in a slightly different way, which may help you understand it better, that is a great resource.

# Onto the App

Finally, after 1400 words of content, we have made it to actually building our app. The app is going to be simple, it will be a todo app. In other words, we are making an app that allows a user to make a todo list, edit tasks in that todo list and complete tasks in the todo list. Why are we making a todo app as an example? Because most web apps need to be able to allow users to create, read, update and delete the data they have access to. This is so important that it even gets a cool acronym: CRUD operations. A todo app is a very simple example where we can easily and intuitively see all of the CRUD operations in action.

We are going to using [Rust](https://www.rust-lang.org/) to create this app. So, get that installed, and perhaps check out [the book](https://doc.rust-lang.org/book/) to get familiarized with the language. I will do my best to explain what's going on every step of the way, but I can't ensure that I'll explain every small aspect of the code we write.

## Creating the Project

Our first step is to create the skeleton of the app, and we'll use Rust's package manager, `cargo`, to do that. Open up your terminal, go to a directory you want to store your project in, and run

```
cargo new todo-app
```

That will create a Rust project within a folder called `todo-app` and will place that folder in your working directory. With that, you can open up the project, and you'll find the following files.

```
todo-app
│   .gitignore
│   Cargo.toml
│
└───src
        main.rs
```

`.gitignore` is not important for our purposes. `Cargo.toml` is where we will be specifying all the libraries we are going to use in our project, and `main.rs` is where our Rust code will go.

## Using Rocket to Make a Web App

With that out of the way, we're going to be installing a web framework called [Rocket ](https://rocket.rs/) to create our web app. Now, the entire goal of this is to make a web app from the ground up, so it seems a bit counter-intuitive to use a framework when building it "from the ground up". However, Rocket is not going to be abstracting away any details that are particularly important to learning more about web apps. It will basically just be handling receiving and sending the HTTP requests and responses we discussed earlier. While we could try and implement the HTTP protocol ourselves, that would be messy and we wouldn't learn much. Further, implementing a protocol is somehow both difficult and tedious, so I'd like to avoid it at all costs.

Now that we're all on the same page, let's set up a basic example. First, in `Cargo.toml`, add Rocket as a dependency. Once you have done that, your `Cargo.toml` should look similar to this

```toml
[package]
name = "todo-app"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]

[dependencies.rocket]
version = "0.5.0-rc.2"
features = ["json"]
```

Now, go to `main.rs`, delete everything in it, and replace it with the following code

```rust
#[macro_use] extern crate rocket;

#[get("/")]
fn index() -> &'static str {
    "Hello, world!"
}

#[launch]
fn rocket() -> _ {
    rocket::build().mount("/", routes![index])
}
```

How does this code work? The first line imports all the stuff we are installing from Rocket, and the `#[macro_use]` means we are explicitly importing Rocket, so it's macros (pieces of code called by writing out a name) are installed globally. `#[get("/")]` is a function attribute being applied to the `index` function. This attribute is imported from Rocket and means that if an HTTP `GET` request is called with the path "/", `index` will be called. `index`, as a function, just returns the string "Hello, world!". The `#[launch]` attribute applied to the `rocket` function means that, when the code is ran, it will start by running the `rocket` function. That launch attribute also does some code magic to set up our server, but we will ignore that for now. Finally, `rocket::build().mount("/", routes![index])` is adding that `index` function we created earlier to the server. Just applying the attirbute to `index` is not enough, we also have to mount `index`.

Let's test it out! Navigate to your project directory in a terminal, and run the following command

```
cargo run
```

You should see a bunch of packages being compiled, and, when it's done, you should get a message along the lines of

```
Rocket has launched from http://127.0.0.1:8000
```

Go to that link, and you'll see a page that says "Hello, world!". Thus, with the install of a package, we have created a basic web app.

If you look in the terminal output, you'll notice something like this listed

```
GET / text/html:
   >> Matched: (index) GET /
   >> Outcome: Success
   >> Response succeeded.
```

As you can see, our server recieved an HTTP `GET` request (`GET` is one of the HTTP methods), and succesfully sent back a response. When we entered the URL into our browser, we sent a `GET` request to the server, the server called the `index` function, and it returned "Hello, world!".

## Creating Tasks

Now, we can get onto the C of our CRUD operations: creation. What we want to do is make it so that when a `POST` request is done on the `/addtask` path with data for a task in its body, the app takes the body data and saves the task somewhere where it will remember.

Why use a `POST` rather than `GET` or one of the many other HTTP request methods? Because `POST` is specifically supposed to be used for requests that result in the creation of data on the server side. Why the path `/addtask`? Because the paths used to do certain operations within the web app should be self-explanatory. When creating a request, you know you are probably creating a task if the path for the request is `/addtask`. Why is the data in the body of the request? Because that's usually where data goes. Headers within the request are normally used for metadata. Parameters and other ways of entering data through the path itself should generally not be used unless you are entering small, simple data.

Obviously, once the tasks are created, we are going to want to store them in some way, so that we can retrieve them to read, update or delete them. However, we don't want to store the data in variables, because if the server is ever turned off, either due to some error or to maintain it, then the data will be lost. Further, the program will only continue to grow in size and become slower over time. As such, we need to use a different solution for longer-term storage of these tasks.

Now, since we are building this from the ground up, like we are the people who made the original web apps and didn't have access to certain tools, we're going to choose a solution that would have made sense at the time. That would be to store the data in a file. A file is something that will exist on the computer even if the program stops or the computer is turned off, and every programming language has access to some form of file creation, edition and deletion.

With that in mind, let's write our add task function. The code added for adding the task looks like this.

```rust
use std::{fs::{OpenOptions}, io::{Write}};
use rocket::serde::{Deserialize, json::Json};

#[derive(Deserialize)]
#[serde(crate = "rocket::serde")]
struct Task<'r> {
    item: &'r str
}

#[post("/addtask", data="<task>")]
fn add_task(task: Json<Task<'_>>) -> &'static str {
    let mut tasks = OpenOptions::new()
                    .read(true)
                    .append(true)
                    .create(true)
                    .open("tasks.txt")
                    .expect("unable to access tasks.txt");   

    let task_item_string = format!("{}\n", task.item);
    let task_item_bytes = task_item_string.as_bytes();

    tasks.write(task_item_bytes).expect("unable to write to tasks.txt");
    "Task added succesfully"
}
```

The attributes in front of the `struct` for `Task` basically say that data pulled from a HTTP JSON body will be put into this struct. The `data="<task>"` in the post attribute signifies that the `Task` struct should specifically be used when deserializing data going to the `/addtask` path. The `OpenOptions` line opens or creates a file called `tasks.txt`, and sets it so that whenever we write to it we are adding rather than overwriting. `task_item_string` and `task_item_bytes` take the data from the `Task` struct and convert it into something that can be written to our file. Finally, we write it to our file and return a string signifying that we have finished the task succesfully.

However, before this works we need to modify our `rocket` function to look like this.

```rust
#[launch]
fn rocket() -> _ {
    rocket::build().mount("/", routes![index, add_task])
}
```

### Testing Our Create Task Method

With that out of the way, we want to test this, but how do we do that? In this tutorial, we are going to use a software called [Postman](https://www.postman.com/) to send HTTP requests to our server, but there are a variety of options for sending requests out there. To use Postman, you'll need to install it from the website, create an account and sign in. After that, you hit `File>New` and choose HTTP request on the page that appears. From there, you want to make your request looks like this.

![](file://C:\Users\garre\AppData\Roaming\marktext\images\2022-06-19-00-47-47-image.png?msec=1656373992260)

You set the method to POST, set the URL to http://127.0.0.1:8000/addtask, and set the body to `raw` with type JSON, and include the following JSON

```json
{
    "item": "Turn into a dragon"
}
```

With that, go back to your terminal, hit `Ctrl-C` to end the server you have running in it, and run `cargo run` again to start the server back up. Once the server is back up, hit Send in Postman. The web app should give an output like this in the terminal:

```
POST /addtask application/json:
   >> Matched: (add_task) POST /addtask
   >> Outcome: Success
   >> Response succeeded.
```

And the response box in Postman should say "Task added succesfully". If you go back to your project directory, you'll notice that a file called `tasks.txt` has been created and has the following content:

```
Turn into a dragon
```

Congratulations! Our webapp can now create tasks.

## Reading Tasks

What's the point in having tasks if we can't get them from the app? That question is rhetorical. There is no point. So, let's make our web app a bit more useful and have it send back all the tasks we have. We'll make so that when a `GET` request is sent to the server with a path of `/readtasks`, we'll send back all the tasks we have stored in `tasks.txt`. We use a `GET` method, because that is the expected method when reading from a server. So, let's skip all the boring stuff and get straight to the code!

First, we need to modify our `task` struct to look like this

```rust
#[derive(Deserialize, Serialize)]
#[serde(crate = "rocket::serde")]
struct Task<'r> {
    item: &'r str
}
```

Now, for the code that actually reads the tasks.

```rust
#[get("/readtasks")]
fn read_tasks() -> Json<Vec<String>> {
    let tasks = OpenOptions::new()
                    .read(true)
                    .append(true)
                    .create(true)
                    .open("tasks.txt")
                    .expect("unable to access tasks.txt");  
    let reader = BufReader::new(tasks);
    Json(reader.lines()
            .map(|line| line.expect("could not read line"))
            .collect())
}
```

Next, we can use postman to send a `GET` request to our server (after stopping, addinng to `read_tasks` to our routes and rerunning), and we can see that we get a list of tasks returned

![](file://C:\Users\garre\AppData\Roaming\marktext\images\2022-06-19-14-01-57-image.png?msec=1656373992261)

## Updating Tasks

Hoo-boy! Now that we've done all that work to create and read tasks, we now have a problem. Editing them, as they currently are, is difficult to impossible. What's the issue? How do we tell the computer what task we want to edit? As of right now, the only way would be to put in the entire original message, but it's not guaranteed that someone won't have two tasks that are written the exact same way. Thus, we will have to store tasks with additional information, and that additional information will be used to identify certain tasks. This will require us to edit both our `add_task` function and our `read_tasks` function, so let's make our edits

### Editing Our Previous Functions

What additional information are we going to add? We'll just put a number next to each task. We'll call it the "id" because it will identify our task. How do we tell where the id ends and the task begins? We'll separate the two via a comma. Then, later, we could save this as a `csv` file and open it in softwares like Excel very easily. With that in mind, let's make our editions to make this happen. First off, in `add_task`, we use a `BufReader` to count the number of lines in the file. The number of lines is the `id` for the new task. It looks like this.

```rust
#[post("/addtask", data="<task>")]
fn add_task(task: Json<Task<'_>>) -> &'static str {
    let mut tasks = OpenOptions::new()
                    .read(true)
                    .append(true)
                    .create(true)
                    .open("tasks.txt")
                    .expect("unable to access tasks.txt");   

    let reader = BufReader::new(&tasks);
    let id = reader.lines().count();

    let task_item_string = format!("{},{}\n", id, task.item);
    let task_item_bytes = task_item_string.as_bytes();

    tasks.write(task_item_bytes).expect("unable to write to tasks.txt");
    "Task added succesfully"
}
```

For `read_tasks`, we don't particularly care for the `id`, so we just discard it, as seen in the code below.

```rust
#[get("/readtasks")]
fn read_tasks() -> Json<Vec<String>> {
    let tasks = OpenOptions::new()
                    .read(true)
                    .append(true)
                    .create(true)
                    .open("tasks.txt")
                    .expect("unable to access tasks.txt");  
    let reader = BufReader::new(tasks);
    Json(reader.lines()
            .map(|line| {
                let line_string: String = line.expect("could not read line");
                let line_pieces: Vec<&str> = line_string.split(",").collect();
                line_pieces[1].to_string()
            })
            .collect())
}
```

Now, you may notice that I had to do quite a bit of work in the `map` function on the last line to make this happen. Let's discuss that quickly.

We use the `split` method to pull apart our original line into multiple pieces. However, the split method splits a `String` into a bunch of string slices `&str`. What that means is instead of having an iterator of a bunch of owned strings, we have an iterator with a bunch of references to the original string. When we collect that iterator into a vector, we have a vector of references (`&str`) rather than a vector of owned strings (`String`). Thus, in the last line, when we pull the piece we want, we use the `to_string()` method to create an owned version of the string slice that we can freely give away to the array that `map` is creating. We can't give away the string slice because it is a reference to a variable that will be deleted once we leave `map`.

In any case, that makes all our edits for our create and read functions, now we can actually make an edit function.

### The Edit Function

This is going to be a `PUT` request put on the path `/edittask`. The code for modifying is a doozy, but here it is

```rust
#[derive(Deserialize, Serialize)]
#[serde(crate = "rocket::serde")]
struct TaskUpdate<'r> {
    id: u8,
    item: &'r str
}

#[put("/edittask", data="<task_update>")]
fn edit_task(task_update: Json<TaskUpdate<'_>>) -> &'static str {
    let tasks = OpenOptions::new()
                    .read(true)
                    .append(true)
                    .create(true)
                    .open("tasks.txt")
                    .expect("unable to access tasks.txt");  
    let mut temp = OpenOptions::new()
                    .create(true)
                    .write(true)
                    .truncate(true)
                    .open("temp.txt")
                    .expect("unable to access temp.txt");

    let reader = BufReader::new(tasks);
    for line in reader.lines() {
        let line_string: String = line.expect("could not read line");
        let line_pieces: Vec<&str> = line_string.split(",").collect();

        if line_pieces[0].parse::<u8>().expect("unable to parse id as u8") == task_update.id {
            let task_items: [&str; 2] = [line_pieces[0], task_update.item];
            let task = format!("{}\n", task_items.join(","));
            temp.write(task.as_bytes()).expect("could not write to temp file");
        }
        else {
            let task = format!("{}\n", line_string);
            temp.write(task.as_bytes()).expect("could not write to temp file");
        }
    }

    std::fs::remove_file("tasks.txt").expect("unable to remove tasks.txt");
    std::fs::rename("temp.txt", "tasks.txt").expect("unable to rename temp.txt");
    "Task updated succesfully"
}
```

As always, you add that to the mounts in the `rocket` function, rerun the program, use postman to send a request, and you can now edit tasks!

Looking at the function, it works in a way that is not completely optimal, but serves our purposes for illustrating using a file to store data. The problem with storing data in a file is that it can be difficult to edit in place. You can use seekers and then write at certain locations, and attempt to delete certain bits, but it generally is a pain. So, instead, we take our file, line by line, and write it to a temporary file `temp.txt`. As we are writing `temp.txt`, we make our modifications, as seen in the `if` statement in our `for...in` loop.

Once we have written `temp.txt`, we delete our original `tasks.txt` and rename `temp.txt` to `tasks.txt`. This will, of course, take longer as our file grows in size, and there is always the worry about getting everything copied properly, but I am not aware of many better solutions. With that completed, we can now move onto deleting tasks.

## Deleting Tasks

Now, we are basically going to use the same exact process that we did with our edit task to make deleting tasks happen. We'll write everything to a temporary file, delete our old file and rename the temporary one. In terms of our actual API, this will make use of a `DELETE` method and will have the `/deletetask` path. Onto some code!

```rust
#[derive(Deserialize, Serialize)]
#[serde(crate = "rocket::serde")]
struct TaskId {
    id: u8
}

#[delete("/deletetask", data="<task_id>")]
fn delete_task(task_id: Json<TaskId>) -> &'static str {
    let tasks = OpenOptions::new()
                    .read(true)
                    .append(true)
                    .create(true)
                    .open("tasks.txt")
                    .expect("unable to access tasks.txt");  
    let mut temp = OpenOptions::new()
                    .create(true)
                    .write(true)
                    .truncate(true)
                    .open("temp.txt")
                    .expect("unable to access temp.txt");

    let reader = BufReader::new(tasks);

    for line in reader.lines() {
        let line_string: String = line.expect("could not read line");
        let line_pieces: Vec<&str> = line_string.split(",").collect();

        if line_pieces[0].parse::<u8>().expect("unable to parse id as u8") != task_id.id {
            let task = format!("{}\n", line_string);
            temp.write(task.as_bytes()).expect("could not write to temp file");
        }
    }

    std::fs::remove_file("tasks.txt").expect("unable to remove tasks.txt");
    std::fs::rename("temp.txt", "tasks.txt").expect("unable to rename temp.txt");
    "Task deleted succesfully"
}
```

Add this to your project and give it a test, and you should notice that we end up with our task deleted. Hooray! With this, we have created all our CRUD operations.

## ID Creation Bug

You probably have noticed that this code still has a few problems. That's fine. Programming is all about coming up with solutions, finding issues with your solutions and making better solutions that account for that. It's an iterative process. Here, we have one big problem right off the bat. Let's say we have three items in `tasks.txt`, so it looks like this

```
0,Buy crab
1,Have existential crisis
2,Eat seafood
```

Now, let's say that we delete the task with id of 1: "Have existential crisis", and we add a new task: "Enjoy sunset". Our `tasks.txt` will now look like this

```
0,Buy crab
2,Eat seafood
2,Enjoy sunset
```

As you can see, two of our tasks now have the same id and will both be targeted when edits or deletes come around. This is a problem with our `add_task` function. Instead of counting lines, we have to go to the last line, which should have the highest id, add 1 to that and use that for the id of our newly created task.

All we have to do to fix this is to modify our `add_task` to the following

```rust
#[post("/addtask", data="<task>")]
fn add_task(task: Json<Task<'_>>) -> &'static str {
    let mut tasks = OpenOptions::new()
                    .read(true)
                    .append(true)
                    .create(true)
                    .open("tasks.txt")
                    .expect("unable to access tasks.txt");   

    let reader = BufReader::new(&tasks);
    let id = match reader.lines().last() {
        None => 0,
        Some(last_line_result) => {
            let last_line = last_line_result.expect("unable to read last line");
            let last_line_pieces: Vec<&str> = last_line.split(",").collect();
            last_line_pieces[0].parse::<u8>().expect("unable to parse integer") + 1
        }
    };

    let task_item_string = format!("{},{}\n", id, task.item);
    let task_item_bytes = task_item_string.as_bytes();

    tasks.write(task_item_bytes).expect("unable to write to tasks.txt");
    "Task added succesfully"
}
```

Which will read our file, go to the last line, and add 1 to the final id, which means our previous example gives an output like this

```
0,Buy crab
2,Eat seafood
3,Enjoy sunset
```

This, of course, means that we have skipped the id of 1, and if we really need every single id we can get, this could prove problematic. In that case, we could make a more sophisticated algorithm that looks for a difference greater than 1 between two lines and uses that to fill out ids, but that is difficult, and, spoiler alert, we are going to be switching out this file based way of saving things for a better solution very shortly, so we are just going to avoid doing anything too fancy.

## Multiple User Issue

We have another issue that I'm not going to address, but this code currently only saves one set of tasks. If this was to work as an actual web app (which we hope to make it at some point), it would also need to be able to handle multiple users using it. I'm sure you can probably guess how that might work though. Assign each person a number, and attach those numbers to tasks in our `tasks.txt`. Then, when we return tasks, we return all the tasks related to a certain user's number. We'll get to this in a minute, too, but I'd like to point out this is possible with files. But, as you probably have noticed, working with files is a huge pain in the ass.

## Error Issue

We have no good errors in place for when we can't find a certain item when editing or deleting. We just always return a success regardless of what happens. Again, since we've already thoroughly experienced the torture of working with a file-based server, we'll get around to this issue once we make our next modification, but it's important to realize that this is another limitation of the code we've put together so far.

## Inputs are Strange

Our `delete_task` and `add_task` functions use an entire JSON input when they take in only one value. As we move along, we may want to make these inputs simpler. Using JSON inputs may be a bit over-engineered.

## Lack of Testing

It's not great to do almost no testing in your code, as it will be harder to maintain, and it's harder to verify that features are working the ways you intend them to. Rust and Rocket both support testing, and we'll get around to doing it, but I want to simplify our large and convoluted codebase before we tackle this.

## Final File-Based Code

Now, I'm going to make a couple of alterations to the code, that I describe below, then we can look at the final `main.rs`

### *Slightly* Better Responses

Our responses will be carefully crafted as we get around to building the frontend of this application, but, as of now, we at least want to give responses that make sense and thus our more conventional. As such, I am going to modify each of the functions to return the task that they modified, deleted, or otherwise dealt with.

### Code Reuse

Things like opening `tasks.txt`, opening `temp.txt` or replacing `tasks.txt` with `temp.txt` get used in multiple places, so I pulled them out into their own functions.

### Improperly Handled Errors

Most of the errors in our code have been handled by `.expect()`. This is generally inadvisable, as this causes the code to panic, which will only show up server side and will leave the user with no response. As such, we are going to return `Result` enums from all of our functions instead, and return the errors. When Rocket encounters a error in a `Result` enum, it will send back an appropriate error response.

However, it's not as simple as that. We do have multiple error types that can occur in our functions, so we actually need to create our own, more generic error type, and implement the functionality for the error to send back the right response. Basically, a bunch of tedious implementation details require us to add this block of code to our final result

```rust
#[derive(Debug)]
enum FileParseError {
    IoError(std::io::Error),
    ParseError(ParseIntError)
}

impl fmt::Display for FileParseError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            Self::IoError(err) => write!(f, "{}", err),
            Self::ParseError(err) => write!(f, "{}", err)
        }
    }
}

impl From<std::io::Error> for FileParseError {
    fn from(error: std::io::Error) -> Self {
        FileParseError::IoError(error)
    }
}

impl From<ParseIntError> for FileParseError {
    fn from(error: ParseIntError) -> Self {
        FileParseError::ParseError(error)
    }
}

impl<'r> Responder<'r, 'r> for FileParseError {
    fn respond_to(self, request: &Request) -> response::Result<'r> {
        match self {
            Self::IoError(err) => {
                err.respond_to(request)
            },
            Self::ParseError(_err) => {
                Err(Status::InternalServerError)
            }
        }
    }
}
```

This will allow us to return IO and integer parsing errors, and to have those errors return proper results when handed off to Rocket. For now, you'll just have to believe me when I say these work.

### The Code

With all of that out of the way, let's look at the entire codebase used to implement our Rust web app so far

```rust
#[macro_use] extern crate rocket;

use core::fmt;
use std::{fs::{OpenOptions, File}, io::{Write, BufReader, BufRead}, num::ParseIntError};
use rocket::{serde::{Deserialize, json::Json, Serialize}, response::{Responder, self}, http::{Status}, Request};

#[derive(Deserialize, Serialize)]
#[serde(crate = "rocket::serde")]
struct Task {
    id: u8,
    item: String
}

#[derive(Deserialize, Serialize)]
#[serde(crate = "rocket::serde")]
struct TaskItem<'r> {
    item: &'r str
}

#[derive(Deserialize, Serialize)]
#[serde(crate = "rocket::serde")]
struct TaskId {
    id: u8
}

#[derive(Debug)]
enum FileParseError {
    IoError(std::io::Error),
    ParseError(ParseIntError)
}

impl fmt::Display for FileParseError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            Self::IoError(err) => write!(f, "{}", err),
            Self::ParseError(err) => write!(f, "{}", err)
        }
    }
}

impl From<std::io::Error> for FileParseError {
    fn from(error: std::io::Error) -> Self {
        FileParseError::IoError(error)
    }
}

impl From<ParseIntError> for FileParseError {
    fn from(error: ParseIntError) -> Self {
        FileParseError::ParseError(error)
    }
}

impl<'r> Responder<'r, 'r> for FileParseError {
    fn respond_to(self, request: &Request) -> response::Result<'r> {
        match self {
            Self::IoError(err) => {
                err.respond_to(request)
            },
            Self::ParseError(_err) => {
                Err(Status::InternalServerError)
            }
        }
    }
}

fn open_tasks_file() -> Result<File, std::io::Error> {
    OpenOptions::new()
                    .read(true)
                    .append(true)
                    .create(true)
                    .open("tasks.txt")
}

fn open_temp_file() -> Result<File, std::io::Error> {
    OpenOptions::new()
                    .create(true)
                    .write(true)
                    .truncate(true)
                    .open("temp.txt")
}

fn save_temp_as_tasks() -> Result<(), std::io::Error> {
    std::fs::remove_file("tasks.txt")?;
    std::fs::rename("temp.txt", "tasks.txt")?;
    Ok(())
}

#[post("/addtask", data="<task>")]
fn add_task(task: Json<TaskItem<'_>>) -> Result<Json<Task>, FileParseError> {
    let mut tasks =  open_tasks_file()?;

    let reader = BufReader::new(&tasks);
    let id = match reader.lines().last() {
        None => 0,
        Some(last_line) => {
            let last_line_parsed = last_line?;
            let last_line_pieces: Vec<&str> = last_line_parsed.split(",").collect();
            last_line_pieces[0].parse::<u8>()? + 1
        }
    };

    let task_item_string = format!("{},{}\n", id, task.item);
    let task_item_bytes = task_item_string.as_bytes();

    tasks.write(task_item_bytes)?;
    Ok(Json(Task {
        id: id,
        item: task.item.to_string()
    }))
}

#[get("/readtasks")]
fn read_tasks() -> Result<Json<Vec<Task>>, FileParseError> {
    let tasks = open_tasks_file()?;
    let reader = BufReader::new(tasks);

    let parsed_lines: Result<Vec<Task>, FileParseError> = reader.lines()
                        .map(|line| {
                            let line_string: String = line?;
                            let line_pieces: Vec<&str> = line_string.split(",").collect();
                            let line_id: u8 = line_pieces[0].parse::<u8>()?;

                            Ok(Task {
                                id: line_id,
                                item: line_pieces[1].to_string()
                            })
                        })
                        .collect();

    Ok(Json(parsed_lines?))
}

#[put("/edittask", data="<task_update>")]
fn edit_task(task_update: Json<Task>) -> Result<Json<Task>, FileParseError> {
    let tasks = open_tasks_file()?; 
    let mut temp = open_temp_file()?;

    let reader = BufReader::new(tasks);
    for line in reader.lines() {
        let line_string: String = line?;
        let line_pieces: Vec<&str> = line_string.split(",").collect();

        if line_pieces[0].parse::<u8>()? == task_update.id {
            let task_items: [&str; 2] = [line_pieces[0], &task_update.item];
            let task = format!("{}\n", task_items.join(","));
            temp.write(task.as_bytes())?;
        }
        else {
            let task = format!("{}\n", line_string);
            temp.write(task.as_bytes())?;
        }
    }

    save_temp_as_tasks()?;
    Ok(task_update)
}

#[delete("/deletetask", data="<task_id>")]
fn delete_task(task_id: Json<TaskId>) -> Result<Json<TaskId>, FileParseError> {
    let tasks = open_tasks_file()?; 
    let mut temp = open_temp_file()?;

    let reader = BufReader::new(tasks);

    for line in reader.lines() {
        let line_string: String = line?;
        let line_pieces: Vec<&str> = line_string.split(",").collect();

        if line_pieces[0].parse::<u8>()? != task_id.id {
            let task = format!("{}\n", line_string);
            temp.write(task.as_bytes())?;
        }
    }

    save_temp_as_tasks()?;
    Ok(task_id)
}

#[get("/")]
fn index() -> &'static str {
    "Hello, world!"
}

#[launch]
fn rocket() -> _ {
    rocket::build().mount("/", routes![index, add_task, read_tasks, edit_task, delete_task])
}
```

Wow! That is a lot of code. That's the thing with a lot of technical fields, though. You keep on iterating over your solutions over and over again until, eventually, you have a really good, but really complicated solution. Nothing we've done throughout this code is too mind melting or big, but, once it's all put together, it looks very big.

However, we can see that a lot of lines are just used to manage our file, and the management of our file has been where most of our design problems and considerations have been going. If there was a way to avoid writing to a file, or to make writing and modifying our data an easier task, our web app could become a lot cleaner, and we could write a lot more features a lot quicker.

# Adding Databases to the App

If the foreshadowing in the last section was not enough, we are going to be removing the file-based saving system in our app, and will replace it with a much more powerful system created specifically for tasks like ours: databases.

Most databases are programs that store tables (this definition is much more limited than the actual definition for databases, but just assume this to be true for our purposes). These tables record information for a variety of individuals in a certain category. For example, you might have a table called `Persons` that stores data on a variety of people. In that example, each row would be a person, and each column would store a different piece of information on that person. The diagram below shows that example.

![An introduction to SQL tables](https://s33046.pcdn.co/wp-content/uploads/2020/07/anatomy-of-a-sql-table-F.png)

The table stores 4 people. For each person, we store their name, last name and age in the columns of our table. The first column, the id, works exactly like the id we used when defining tasks. The id is a simple and easy way to access a certain person.

Now, unlike our file example, databases make it very easy to perform CRUD operations on these tables. Creating, Reading, Updating and Deleting certain entries of the table are handled via simple commands, so we don't have to worry about the overhead of writing, renaming and deleting files. It also comes with a variety of other advantages that will show up naturally as we continue to write our web app.

But, now that we're aware of what a database is, let's try and use one.

## PostgreSQL

For our purposes, [PostgreSQL](https://www.postgresql.org/) will serve just fine. There are many different databases out there, all with their own pros and cons, so if you end up working on your own software, keep that in mind and do your research to determine the best option. In any case, install Postgres. Have it listen on port `5432` and when it asks for a password for your database just list that as `password` for now.

If you are having trouble installing or using postgres on Windows, you can check out the following video from Amigoscode: [Getting Started with PostgreSQL for Windows | 2021](https://www.youtube.com/watch?v=BLH3s5eTL4Y). Sadly, I run a Windows machine, so I can't vouch for any Mac or Linux resources for setting up postgres, but there are several online.

Now, let's use our newly installed database! Run `psql`, which is the SQL Shell you installed to use postgres. Log into your database with the following information:

- Server: `localhost`
  
- Database: `postgres`
  
- Port: `5432`
  
- Username: `postgres`
  
- Password: `password`
  

This will connect you to the postgres database. Now, we're going to use this connection to create a new database to test out the CRUD operations available to us. Simply run the following command:

```sql
CREATE DATABASE test;
```

Note that the semicolon is important, because postgres will not view the command as "ready to use" until a semicolon is found. With that, if you run `\l`, you should get the following output.

```
                                                 List of databases
   Name    |  Owner   | Encoding |          Collate           |           Ctype            |   Access privileges
-----------+----------+----------+----------------------------+----------------------------+-----------------------
 postgres  | postgres | UTF8     | English_United States.1252 | English_United States.1252 |
 template0 | postgres | UTF8     | English_United States.1252 | English_United States.1252 | =c/postgres          +
           |          |          |                            |                            | postgres=CTc/postgres
 template1 | postgres | UTF8     | English_United States.1252 | English_United States.1252 | =c/postgres          +
           |          |          |                            |                            | postgres=CTc/postgres
 test      | postgres | UTF8     | English_United States.1252 | English_United States.1252 |
(4 rows)
```

As you can see, postgres came with 3 databases pre-packaged: `postgres`, `template0` and `template1`. The 4th database, `test`, is the one we have created. Now, run `quit` to leave the shell, and re-enter the shell. Login the same way as last time, but list `test` as your database rather than `postgres`. With that, you'll log into your newly created database.

Now, to test out our CRUD operations, we have to create a table, so run the following command (by the way, since postgres doesn't process a command until it reaches a semicolon, you can write commands out over multiple lines).

```sql
CREATE TABLE person (
    id BIGSERIAL NOT NULL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    country VARCHAR(50) NOT NULL
);
```

With this, you'll create a table. This table will take items that have a name and country listed, and the id is a number that will be incremented automatically whenever we create a new entry. If you want to see all the relations in your database (which includes your table and the auto incrementing id), you can run `\d`. If you want to see all the tables in your database (which is just person currently) run `\dt`. The output of `\d`, if everything has been done correctly, should look like this:

```
              List of relations
 Schema |     Name      |   Type   |  Owner
--------+---------------+----------+----------
 public | person        | table    | postgres
 public | person_id_seq | sequence | postgres
(2 rows)
```

Now, let's do some CRUD operations! First off, let's do the creation operation. Run the following commands to add a few people to your person table.

```sql
INSERT INTO person (name, country) VALUES ('Torvald', 'Norway');
INSERT INTO person (name, country) VALUES ('Abelarda', 'Germany');
INSERT INTO person (name, country) VALUES ('Melete', 'Italy');
```

Next, let's do a read operation. Run the following command to look at all the new entries to your table.

```sql
SELECT * FROM person
```

The output should look like the following

```
id  |   name   | country
----+----------+---------
  1 | Torvald  | Norway
  2 | Abelarda | Germany
  3 | Melete   | Italy
(3 rows)
```

Now, we can do our update operation by running the following command.

```sql
UPDATE person SET country = 'USA' WHERE id = 3;
```

This will make Melete's country the US.

Finally, let's do the delete operation. Just run the following command to remove Abelarda from our table

```sql
DELETE FROM person WHERE id = 2;
```

If you exit and re-enter the database at any point between these operations, and even if you turn off the computer, your database will still hold onto these values. Thus, we were able to achieve persistent CRUD operations via minimal effort. I'm sure you probably would agree that the commands and the database are easier to use and manage than a file, even if both are attempting to solve the same problem.

However, just because we have this database means nothing if we can't use it with our app. That's where database drivers come into play.

## Database Drivers

A database driver solves the exact problem we have. It allows you to connect, and use a database via a program. Instead of logging in and running the operations on our database manually, like we did in the previous section, we instead use a database driver to connect to the database, and then write out our operations in our program. These database drivers are generally libraries that you install for your project, the same way we installed Rocket.

However, in our case, Rocket requires us to use one of their supported database drivers for our application. But, regardless of what we're using, let's setup the driver. By the way, most of the code here will be *a little* shoddy, because we are eventually going to replace this with something slightly more sophisticated that will result in less code.

In any case, modify your `Cargo.toml` to have the following imports

```toml
[package]
name = "todo-app"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]

[dependencies.rocket]
version = "0.5.0-rc.2"
features = ["json"]

[dependencies.sqlx]
version = "0.5"
default-features = false
features = ["macros", "offline", "migrate"]

[dependencies.rocket_db_pools]
version = "0.1.0-rc.2"
features = ["sqlx_postgres"]
```

Next, go into `main.rs` and add the following code next to all your other structs.

```rust
#[derive(Database)]
#[database("todo")]
struct TodoDatabase(sqlx::PgPool);
```

Then, at the same level as the `Cargo.toml`, create a file called `Rocket.toml` and enter the following code

```toml
[default.databases.todo]
url = "postgres://postgres:password@localhost:5432/todo"
```

Then, go to the `rocket` function in `main.rs` and modify it to look like the following. This will intialize our database.

```rust
#[launch]
fn rocket() -> _ {
    rocket::build()
        .attach(TodoDatabase::init())
        .mount("/", routes![index, add_task, read_tasks, edit_task, delete_task])
}
```

Now, when we implement the CRUD operations, we will assume that we have a table called `tasks` in our database. The next step is to create our database (which we have named `todo` based on our configuration) and the `tasks` table in postgres. Simply head over to `psql` and run the following command

```sql
CREATE DATABASE todo;
```

This will create a database called `todo`. Now, connect to `todo` and run this command

```sql
CREATE TABLE IF NOT EXISTS tasks (
    id   BIGSERIAL NOT NULL PRIMARY KEY,
    item TEXT NOT NULL
);
```

This will create our `tasks` table in our database.

## CRUD Operations with a Database

Now that that is done, we can change our CRUD operations to use our database rather than a file. We get access to the database by just including an argument in our method, and then we can use that variable and various functions to run the commands that we were running earlier. So, let's start modifying our functions. The first thing we are going to do is add the creation operation, and we also add in a little bit of code to make the errors work. I also made some slight adjustments to the `Task` and `TaskId` struct to make them play nice with the database driver, so implement those changes as well.

If you're wondering about the error code, I'm just using a tuple struct to wrap `rocket_db_pools::sqlx::Error`, so I can implement the `Responder` trait for it. I also have to implement the `From` trait for the error, so that the two can be converted between nicely.

```rust
#[derive(Deserialize, Serialize, sqlx::FromRow)]
#[serde(crate = "rocket::serde")]
struct Task {
    id: i64,
    item: String
}

#[derive(Deserialize, Serialize)]
#[serde(crate = "rocket::serde")]
struct TaskItem<'r> {
    item: &'r str
}

#[derive(Deserialize, Serialize)]
#[serde(crate = "rocket::serde")]
struct TaskId {
    id: i64
}

struct DatabaseError(rocket_db_pools::sqlx::Error);

impl<'r> Responder<'r, 'r> for DatabaseError {
    fn respond_to(self, request: &Request) -> response::Result<'r> {
        Err(Status::InternalServerError)
    }
}

impl From<rocket_db_pools::sqlx::Error> for DatabaseError {
    fn from(error: rocket_db_pools::sqlx::Error) -> Self {
        DatabaseError(error)
    }
}

#[post("/addtask", data="<task>")]
async fn add_task(task: Json<TaskItem<'_>>, mut db: Connection<TodoDatabase>) -> Result<Json<Task>, DatabaseError> {
    let added_task = sqlx::query_as::<_, Task>("INSERT INTO tasks (item) VALUES ($1) RETURNING *")
        .bind(task.item)
        .fetch_one(&mut *db)
        .await?;

    Ok(Json(added_task))
}
```

As you can see, by avoiding files and using a database instead, we have greatly simplified the creation operation. Most of our code is just making the database driver send the proper request (the stuff after `added_task`) and making sure our errors respond properly (everything above `add_task`), which is pretty wonderful. With that out of the way, let's now implement the read operation

The read operation is also simple, we just use that `SELECT` command we used earlier to grab all the available results.

```rust
#[get("/readtasks")]
async fn read_tasks(mut db: Connection<TodoDatabase>) -> Result<Json<Vec<Task>>, DatabaseError> {
    let all_tasks = sqlx::query_as::<_, Task>("SELECT * FROM tasks")
        .fetch_all(&mut *db)
        .await?;

    Ok(Json(all_tasks))
}
```

Of course, the update operation simply uses the `UPDATE` command we discussed earlier, and, just like the creation operation, we use the `RETURNING` statement to return our actual modified task, so we can return it to the user.

```rust
#[put("/edittask", data="<task_update>")]
async fn edit_task(task_update: Json<Task>, mut db: Connection<TodoDatabase>) -> Result<Json<Task>, DatabaseError> {
    let updated_task = sqlx::query_as::<_, Task>("UPDATE tasks SET item = $1 WHERE id = $2 RETURNING *")
        .bind(&task_update.item)
        .bind(task_update.id)
        .fetch_one(&mut *db)
        .await?;

    Ok(Json(updated_task))
}
```

Finally, the `DELETE` operation is the same story. Take the command we used before, shove it into the database driver and let it run

```rust
#[delete("/deletetask", data="<task_id>")]
async fn delete_task(task_id: Json<TaskId>, mut db: Connection<TodoDatabase>) -> Result<Json<Task>, DatabaseError> {
    let deleted_task = sqlx::query_as::<_, Task>("DELETE FROM tasks WHERE id = $1 RETURNING *")
        .bind(task_id.id)
        .fetch_one(&mut *db)
        .await?;

    Ok(Json(deleted_task))
}
```

With this, we have persistent CRUD operations using MUCH LESS code. After deleting all the redundant/unneeded code, we have a code base that clocks in at less than 100 lines.

```rust
#[macro_use] extern crate rocket;

use rocket::{serde::{Deserialize, json::Json, Serialize}, response::{Responder, self}, http::{Status}, Request};
use rocket_db_pools::{Database, Connection};

#[derive(Deserialize, Serialize, sqlx::FromRow)]
#[serde(crate = "rocket::serde")]
struct Task {
    id: i64,
    item: String
}

#[derive(Deserialize, Serialize)]
#[serde(crate = "rocket::serde")]
struct TaskItem<'r> {
    item: &'r str
}

#[derive(Deserialize, Serialize)]
#[serde(crate = "rocket::serde")]
struct TaskId {
    id: i64
}

#[derive(Database)]
#[database("todo")]
struct TodoDatabase(sqlx::PgPool);

struct DatabaseError(rocket_db_pools::sqlx::Error);

impl<'r> Responder<'r, 'r> for DatabaseError {
    fn respond_to(self, _request: &Request) -> response::Result<'r> {
        Err(Status::InternalServerError)
    }
}

impl From<rocket_db_pools::sqlx::Error> for DatabaseError {
    fn from(error: rocket_db_pools::sqlx::Error) -> Self {
        DatabaseError(error)
    }
}

#[post("/addtask", data="<task>")]
async fn add_task(task: Json<TaskItem<'_>>, mut db: Connection<TodoDatabase>) -> Result<Json<Task>, DatabaseError> {
    let added_task = sqlx::query_as::<_, Task>("INSERT INTO tasks (item) VALUES ($1) RETURNING *")
        .bind(task.item)
        .fetch_one(&mut *db)
        .await?;

    Ok(Json(added_task))
}

#[get("/readtasks")]
async fn read_tasks(mut db: Connection<TodoDatabase>) -> Result<Json<Vec<Task>>, DatabaseError> {
    let all_tasks = sqlx::query_as::<_, Task>("SELECT * FROM tasks")
        .fetch_all(&mut *db)
        .await?;

    Ok(Json(all_tasks))
}

#[put("/edittask", data="<task_update>")]
async fn edit_task(task_update: Json<Task>, mut db: Connection<TodoDatabase>) -> Result<Json<Task>, DatabaseError> {
    let updated_task = sqlx::query_as::<_, Task>("UPDATE tasks SET item = $1 WHERE id = $2 RETURNING *")
        .bind(&task_update.item)
        .bind(task_update.id)
        .fetch_one(&mut *db)
        .await?;

    Ok(Json(updated_task))
}

#[delete("/deletetask", data="<task_id>")]
async fn delete_task(task_id: Json<TaskId>, mut db: Connection<TodoDatabase>) -> Result<Json<Task>, DatabaseError> {
    let deleted_task = sqlx::query_as::<_, Task>("DELETE FROM tasks WHERE id = $1 RETURNING *")
        .bind(task_id.id)
        .fetch_one(&mut *db)
        .await?;

    Ok(Json(deleted_task))
}

#[get("/")]
fn index() -> &'static str {
    "Hello, world!"
}

#[launch]
fn rocket() -> _ {
    rocket::build()
        .attach(TodoDatabase::init())
        .mount("/", routes![index, add_task, read_tasks, edit_task, delete_task])
}
```

However, programming is iterative. There are still improvements that could be made to this codebase, and we are going to make them. The first is cutting down the boilerplate code even more via Object-Relational Mapping, an ORM.

## ORMs

Obect-Relational Mapping is a technique for converting data between type systems in object-oriented programming languages. To simplify, it is a technique for converting classes and their data into data that can be used by other programs. Generally, though, when some mentions "an ORM", they are discussing a library that uses this technique.

ORMs are almost always used with databases. An ORM will take an object in a class, or an instance of a struct, and create an entry in a database for it. So, we could just feed an ORM an instance of our `Task` struct, and it will create an entry in the `tasks` table for us, rather than us being required to write the `sql` ourselves.

So, an ORM is basically a way to reduce boilerplate code. Rather than writing a bunch of repetitive `sql` code, we leave it to the ORM to generate the `sql` code based on our structs and the annotations we leave within those structs. Also, ORMs often implement ways to manage creating the tables available in your database. These are called "migrations".

Basically, our database driver gives us the ability to run `sql` commands on our database. It simply allows to interact with our database. An ORM allows to write in the language of our choice, rather than running `sql` commands to interact with the database.

With the way we've configured Rocket, it is expecting to use an asynchronous ORM. The most popular async ORM in Rust at the moment is [SeaQL/sea-orm](https://github.com/SeaQL/sea-orm), so this is what we'll be implementing on our code base.

First, modify your `Cargo.toml` to look like the following

```toml
[package]
name = "todo-app"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[workspace]
members = [".", "entity", "migration"]

[dependencies]
async-trait = { version = "0.1" }
entity = { path = "entity" }
migration = { path = "migration" }

[dependencies.rocket]
version = "0.5.0-rc.2"
features = ["json"]

[dependencies.sea-orm-rocket]
git = "https://github.com/SeaQL/sea-orm"

[dependencies.sea-orm]
version = "^0.8.0"
features = [
  "runtime-tokio-native-tls",
  "sqlx-postgres",
  "macros"
]
```

With that, we have installed `sea-orm` and some other libraries we'll need, and have removed the libraries that are now redundant. Now, the first thing we are going to do is create those migrations I mentioned before. We'll use `sea-orm` to create the tables in our database, so we don't have to do that by hand.

### Migrations

A t your terminal, run the following command

```shell
cargo install sea-orm-cli
```

This will install a command line interface (cli), so we can run certain commands at our terminal. As you can imagine, these commands make it easier to do certain things with `sea-orm`. Next, we are going to create the directory to hold our migrations by running the following command.

```shell
sea-orm-cli migrate init
```

In my current version of `sea-orm`, this generates a slightly wrong `Cargo.toml`. Ensure that the `Cargo.toml` in your `migration` directory has `async-std` imported like the following

```toml
[package]
name = "migration"
version = "0.1.0"
edition = "2021"
publish = false

[lib]
name = "migration"
path = "src/lib.rs"

[dependencies]
entity = { path = "../entity" }
rocket = { version = "0.5.0-rc.1" }

[dependencies.sea-orm-migration]
version = "^0.8.0"
features=["sqlx-postgres", "runtime-tokio-native-tls"]

[dependencies.async-std]
version = "^1"
features = ["attributes", "tokio1"]
```

Now, we'll create a new migration by running the following command

```shell
sea-orm-cli migrate generate create_tasks_table
```

Now, if you go to `migration/src`, you'll see a file that starts with `m`, has some numbers and has `create_tasks_table` at the end. For me, it ended up being `m20220623_084419_create_tasks_table`, but the number is time dependent so it will be different for you. Regardless, open that file. Now, in that file put the following code (NOTE: under `impl MigrationName...`, keep the string the same as the name of the file. It should not be the name I have)

```rust
use sea_orm_migration::prelude::*;

pub struct Migration;

impl MigrationName for Migration {
    fn name(&self) -> &str {
        "m20220622_210301_create_tasks_table"
    }
}

#[async_trait::async_trait]
impl MigrationTrait for Migration {
    async fn up(&self, manager: &SchemaManager) -> Result<(), DbErr> {
        manager
            .create_table(
                sea_query::Table::create()
                    .table(Task::Table)
                    .if_not_exists()
                    .col(
                        ColumnDef::new(Task::Id)
                            .integer()
                            .not_null()
                            .auto_increment()
                            .primary_key(),
                    )
                    .col(ColumnDef::new(Task::Item).string().not_null())
                    .to_owned()
            )
            .await
    }

    async fn down(&self, manager: &SchemaManager) -> Result<(), DbErr> {
        manager
            .drop_table(
                sea_query::Table::drop()
                    .table(Task::Table)
                    .to_owned()
            )
            .await
    }
}

#[derive(Iden)]
pub enum Task {
    Table,
    Id,
    Item
}
```

Delete the other file with the numbers at the front and modify `lib.rs` so it only uses your migration like this

```rust
pub use sea_orm_migration::prelude::*;

mod m20220622_210301_create_tasks_table;

pub struct Migrator;

#[async_trait::async_trait]
impl MigratorTrait for Migrator {
    fn migrations() -> Vec<Box<dyn MigrationTrait>> {
        vec![
            Box::new(m20220622_210301_create_tasks_table::Migration),
        ]
    }
}
```

Next, make sure the `main.rs` in `migration/src` looks like the following.

```rust
use sea_orm_migration::prelude::*;

#[async_std::main]
async fn main() {
    //  Setting `DATABASE_URL` environment variable
    let key = "DATABASE_URL";
    if std::env::var(key).is_err() {
        // Getting the database URL from Rocket.toml if it's not set
        let figment = rocket::Config::figment();
        let database_url: String = figment
            .extract_inner("databases.todo.url")
            .expect("Cannot find Database URL in Rocket.toml");
        std::env::set_var(key, database_url);
    }

    cli::run_cli(migration::Migrator).await;
}
```

Now, at the terminal run

```shell
cargo new entity
```

Make sure `entity`'s `Cargo.toml` looks like the following

```toml
[package]
name = "entity"
version = "0.1.0"
edition = "2021"

[lib]
name = "entity"
path = "src/lib.rs"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]

[dependencies.rocket]
version = "0.5.0-rc.2"
features = ["json"]

[dependencies.sea-orm]
version = "^0.8.0"
```

Note that the `name` and `path` values underneath the `[lib]` are default values. But those are just put there in case the defaults ever change. All we really need is the `[lib]` to declare that we Cargo has a library target.

Rename `entity/src/main.rs` to `lib.rs`. We'll make sure that this gets replaced with some code that can be imported in a bit.

Over at our original `src`, `sea-orm` is going to be replacing the database driver we used in our example up until now. It's the same database driver for the most part, but it has some added features and other peculiar things to make it work with `sea-orm`. For now, we are just going to treat the code that allows us to connect to our database as boilerplate that doesn't need to be understood. We'll be putting this boilerplate in it's own file, and then importing the needed parts into `main.rs`. So, in `src`, create a new file called `pool.rs` with the following code

```rust
use async_trait::async_trait;
use sea_orm::ConnectOptions;
use sea_orm_rocket::{rocket::figment::Figment, Config, Database};
use std::time::Duration;

#[derive(Database, Debug)]
#[database("todo")]
pub struct Db(SeaOrmPool);

#[derive(Debug, Clone)]
pub struct SeaOrmPool {
    pub conn: sea_orm::DatabaseConnection,
}

#[async_trait]
impl sea_orm_rocket::Pool for SeaOrmPool {
    type Error = sea_orm::DbErr;

    type Connection = sea_orm::DatabaseConnection;

    async fn init(figment: &Figment) -> Result<Self, Self::Error> {
        let config = figment.extract::<Config>().unwrap();
        let mut options: ConnectOptions = config.url.into();
        options
            .max_connections(config.max_connections as u32)
            .min_connections(config.min_connections.unwrap_or_default())
            .connect_timeout(Duration::from_secs(config.connect_timeout));
        if let Some(idle_timeout) = config.idle_timeout {
            options.idle_timeout(Duration::from_secs(idle_timeout));
        }
        let conn = sea_orm::Database::connect(options).await?;

        Ok(SeaOrmPool { conn })
    }

    fn borrow(&self) -> &Self::Connection {
        &self.conn
    }
}
```

Now, we are going to change our main function to connect to our newer, fancier database and to run the migrations that we are creating. Thus, go into `main.rs` and change the `rocket` function to look like the following.

```rust
async fn run_migrations(rocket: Rocket<Build>) -> fairing::Result {
    let conn = &Db::fetch(&rocket).unwrap().conn;
    let _ = migration::Migrator::up(conn, None).await;
    Ok(rocket)
}

#[launch]
fn rocket() -> _ {
    rocket::build()
        .attach(Db::init())
        .attach(AdHoc::try_on_ignite("Migrations", run_migrations))
        .mount("/", routes![index])
}
```

Now, it hurts to do this, but none of this code in `main.rs` is purposeful anymore, so we are going to delete it. Delete the `Task` struct, `TaskItem` struct, `TaskId` struct, `TodoDatabase` struct, `DatabaseError` struct, the two implementations for `Database Error`, `add_task`, `read_tasks`, `edit_task` and `delete_task`. When all is said and done, our `main.rs` is back to almost being new

```rust
#[macro_use] extern crate rocket;

mod pool;


use migration::MigratorTrait;
use pool::Db;
use rocket::{fairing::{AdHoc, self}, Rocket, Build};
use sea_orm_rocket::{Database};

#[get("/")]
fn index() -> &'static str {
    "Hello, world!"
}

async fn run_migrations(rocket: Rocket<Build>) -> fairing::Result {
    let conn = &Db::fetch(&rocket).unwrap().conn;
    let _ = migration::Migrator::up(conn, None).await;
    Ok(rocket)
}

#[launch]
fn rocket() -> _ {
    rocket::build()
        .attach(Db::init())
        .attach(AdHoc::try_on_ignite("Migrations", run_migrations))
        .mount("/", routes![index])
}
```

Now, over in Postgres, log into the `todo` database and delete our pre-existing `tasks` table with the following command

```sql
DROP TABLE tasks;
```

With all of that out of the way, you can once again run `cargo run` in the terminal, which will use our migration to create a `tasks` table. It will also create a table called `seaql_migrations`, which will keep track of information on our migrations.

### Entities

Entities are the fancy structs that we'll use to pull data from and put data in to the database. With SeaORM, there is the option of genearting entities from our database after running our migrations. But, it's usually better to only ever run this once and write most of your entities by hand. Mainly because we'll be editing these entities to work better in our project, and running the entity generation command overwrites the changes we've made. As such, I'm just going to give you the code for the entities we make, but keep in mind you can generate entities from an existing database if you ever find a need or a good use for it.

So, in `entity/src` create a file called `tasks.rs` and enter the following code

```rust
use rocket::{serde::{Deserialize, Serialize}, FromForm};
use sea_orm::entity::prelude::*;

#[derive(Clone, Debug, PartialEq, DeriveEntityModel, Deserialize, Serialize, FromForm)]
#[serde(crate = "rocket::serde")]
#[sea_orm(table_name = "tasks")]
pub struct Model {
    #[sea_orm(primary_key)]
    #[field(default = 0)]
    pub id: i32,
    pub item: String,
}

#[derive(Copy, Clone, Debug, EnumIter)]
pub enum Relation {}

impl RelationTrait for Relation {
    fn def(&self) -> RelationDef {
        panic!("No RelationDef")
    }
}

impl ActiveModelBehavior for ActiveModel {}
```

By the way, you may notice that our `id` field has the attribute `#[field(default = 0)]` applied to it. This makes it so, when our data is processed, the `id` will default to 0 if no value is given, so `Task`s can be entered with no `id`.

So, in terms of our code, we can now use our exported `tasks` model as the struct for getting entered data. Note that the model implements `FromForm`, so we will be entering our data as `x-www-form-urlencoded` now. Considering are small amount of basic parameters, this switch makes a lot of sense. Further, in Rocket, form parsing is lenient by default, so if there is missing, duplicate or extra fields, it will still parse. Considering we have places where we may get an id, but no item; an item, but no id; or both, this will reduce the extra code we had to write before. In any case, let's re-implement all our CRUD operations via the ORM. You'll notice that I am mainly using the model to find items in the database, and to add items I've put together using the model's struct into the database. That's the power of an ORM, I don't have to write `sql` now, I can just write out functions in Rust.

So, let's get started! First is re-writing the creation operation. Here's the code for that

```rust
struct DatabaseError(sea_orm::DbErr);

impl<'r> Responder<'r, 'r> for DatabaseError {
    fn respond_to(self, _request: &Request) -> response::Result<'r> {
        Err(Status::InternalServerError)
    }
}

impl From<sea_orm::DbErr> for DatabaseError {
    fn from(error: sea_orm::DbErr) -> Self {
        DatabaseError(error)
    }
}

#[post("/addtask", data="<task_form>")]
async fn add_task(conn: Connection<'_, Db>, task_form: Form<tasks::Model>) -> Result<Json<tasks::Model>, DatabaseError> {
    let db = conn.into_inner();
    let task = task_form.into_inner();

    let active_task: tasks::ActiveModel = tasks::ActiveModel {
        item: Set(task.item),
        ..Default::default()
    };

    Ok(Json(active_task.insert(db).await?))
}
```

As you can see, we've once again used `DatabaseError` as a wrapper around the actual error from our database library, so that we can return those errors. In `add_task`, the `into_inner` functions pull the data we want from the variables we have. Finally, in SeaORM, to use a model to update or create an item, it needs to be an `ActiveModel`, so we have to use the data we already have to create an `ActiveModel` version. Finally, the last line is, of course, inserting the item into the database.

Next is the read operation which is even easier. I've included the import for one of the variables used in this function, because it is a named import, and you wouldn't be able to import it properly otherwise.

```rust
use entity::tasks::Entity as Tasks;

#[get("/readtasks")]
async fn read_tasks(conn: Connection<'_, Db>) -> Result<Json<Vec<tasks::Model>>, DatabaseError> {
    let db = conn.into_inner();

    Ok(Json(
        Tasks::find()
            .order_by_asc(tasks::Column::Id)
            .all(db)
            .await?
    ))
}
```

This one is pretty simple as well. Us the database to find all tasks and order them by ascending ids.

Next, `edit_task` is a bit more complicated, but it still is pretty simple. We find the task we are attempting to update, `task_to_update`, modify the fields we would like to on it, then call `update` to update the database.

```rust
#[put("/edittask", data="<task_form>")]
async fn edit_task(conn: Connection<'_, Db>, task_form: Form<tasks::Model>) -> Result<Json<tasks::Model>, DatabaseError> {
    let db = conn.into_inner();
    let task = task_form.into_inner();

    let task_to_update = Tasks::find_by_id(task.id).one(db).await?;
    let mut task_to_update: tasks::ActiveModel = task_to_update.unwrap().into();
    task_to_update.item = Set(task.item);

    Ok(Json(
        task_to_update.update(db).await?
    ))
}
```

Finally, `delete_task` is the only one that has had its API changed a little bit. Due to the fact that I only get a `DeleteResult` from SeaORM, I now just return how many tasks were deleted rather than the deleted task. Also, because I can't ensure that an `id` is given with a task form, and the only value I'm taking in is an `id`, I make `id` a parameter of the url, which works nicely in this case. So, it looks different but basically does the same thing

```rust
#[delete("/deletetask/<id>")]
async fn delete_task(conn: Connection<'_, Db>, id: i32) -> Result<String, DatabaseError> {
    let db = conn.into_inner();
    let result = Tasks::delete_by_id(id).exec(db).await?;

    Ok(format!("{} task(s) deleted", result.rows_affected))
}
```

Thus, the overall `main.rs` looks like this

```rust
#[macro_use] extern crate rocket;

mod pool;


use migration::MigratorTrait;
use pool::Db;
use rocket::{fairing::{AdHoc, self}, Rocket, Build, form::Form, serde::json::Json, http::Status, response::{Responder, self}, Request};
use sea_orm::{ActiveModelTrait, Set, EntityTrait, QueryOrder, DeleteResult};
use sea_orm_rocket::{Database, Connection};

use entity::tasks;
use entity::tasks::Entity as Tasks;

struct DatabaseError(sea_orm::DbErr);

impl<'r> Responder<'r, 'r> for DatabaseError {
    fn respond_to(self, _request: &Request) -> response::Result<'r> {
        Err(Status::InternalServerError)
    }
}

impl From<sea_orm::DbErr> for DatabaseError {
    fn from(error: sea_orm::DbErr) -> Self {
        DatabaseError(error)
    }
}

#[post("/addtask", data="<task_form>")]
async fn add_task(conn: Connection<'_, Db>, task_form: Form<tasks::Model>) -> Result<Json<tasks::Model>, DatabaseError> {
    let db = conn.into_inner();
    let task = task_form.into_inner();

    let active_task: tasks::ActiveModel = tasks::ActiveModel {
        item: Set(task.item),
        ..Default::default()
    };

    Ok(Json(active_task.insert(db).await?))
}

#[get("/readtasks")]
async fn read_tasks(conn: Connection<'_, Db>) -> Result<Json<Vec<tasks::Model>>, DatabaseError> {
    let db = conn.into_inner();

    Ok(Json(
        Tasks::find()
            .order_by_asc(tasks::Column::Id)
            .all(db)
            .await?
    ))
}

#[put("/edittask", data="<task_form>")]
async fn edit_task(conn: Connection<'_, Db>, task_form: Form<tasks::Model>) -> Result<Json<tasks::Model>, DatabaseError> {
    let db = conn.into_inner();
    let task = task_form.into_inner();

    let task_to_update = Tasks::find_by_id(task.id).one(db).await?;
    let mut task_to_update: tasks::ActiveModel = task_to_update.unwrap().into();
    task_to_update.item = Set(task.item);

    Ok(Json(
        task_to_update.update(db).await?
    ))
}

#[delete("/deletetask/<id>")]
async fn delete_task(conn: Connection<'_, Db>, id: i32) -> Result<String, DatabaseError> {
    let db = conn.into_inner();
    let result = Tasks::delete_by_id(id).exec(db).await?;

    Ok(format!("{} task(s) deleted", result.rows_affected))
}

#[get("/")]
fn index() -> &'static str {
    "Hello, world!"
}

async fn run_migrations(rocket: Rocket<Build>) -> fairing::Result {
    let conn = &Db::fetch(&rocket).unwrap().conn;
    let _ = migration::Migrator::up(conn, None).await;
    Ok(rocket)
}

#[launch]
fn rocket() -> _ {
    rocket::build()
        .attach(Db::init())
        .attach(AdHoc::try_on_ignite("Migrations", run_migrations))
        .mount("/", routes![index, add_task, read_tasks, edit_task, delete_task])
}
```

And, with that, we have once again implemented our CRUD operations, this time using an ORM. However, we have added a lot of code to the code base, and have only saved approximately 3 lines in `main.rs`. Let's talk about that.

### Over-engineering a Problem

With any software problem, we can add complexity to increase flexibility. In other words, we can add more code and make our solution more complicated, so that it's easier to change certain aspects of our solution or to add new features to it. Generally, we want our code to be somewhat flexible. Since programming is iterative, it is almost assured that someone will look at something you wrote, and decide to change pieces of it or to add new features to it. If your code is flexible, whatever developer has to do this will have a much easier time.

However, increased complexity makes it harder to understand your code. Further, the more complex a solution is, the more prone it is to bugs. Little problems or errors that are hard to expect or notice. On top of that, more complex code is harder to maintain. There's more code to maintain, the code is doing fancier things that have to be more carefullly maintained, and, assuming this more complex code is using more functions or features from libraries, it is more liable to break when a certain library updates.

Thus, with any solution you have to be careful. A very flexible solution will also be very complex, and will be a nightmare to work with, even though the original intention was to make it easier to work with. On the other hand, a very simple solution will have no flexibility, so it will probably have to be re-written from the ground up whenever a new feature needs to be added. Neither extreme is ideal.

When you "over-engineer" a problem, it means that you have made your code far too flexible, and, in the process, made it much more complex than it should be. Of course, if a solution is actually over-engineered or not is a matter of subjective opinion. It depends on the tastes and sensibilities of the developer. But, just like any subjective thing, there are certain cases where most people agree. I believe most people would agree that implementing an ORM for the project we are doing is over-engineering.

An ORM makes it super easy to create large amounts of tables with a lot of relationships, and to work with the data inside that large database. It's a tool for when you have 10s or 100s of tables. In our case, we are only going to have (approximately) 2 tables by the time we're done with this app. While maintaining them directly via `sql` is more difficult, it also requires a lot less code complexity, and is manageable for a small amount of tables.

So, why did I implement an ORM at all? As I stated at the start of this piece, my goal is to show off all the sort of libraries and concepts that modern web apps and frameworks use, so you can get a better understanding of them. While an ORM is not a great idea for this app, it is a concept I wanted to demonstrate, because ORMs are pretty much ubiquitous in backend frameworks. There are going to be many other things where the solution ends up adding a lot of complexity for very little benefit to the app, and that's because I'm trying to demonstrate the concepts used in modern web dev, even if not all of them are perfect for the particular app we are making.

## Containerization

You may have noticed an interesting issue with our current database. That's the fact that we have to install and setup an application on our machine to make our code work. When only a single developer is working on a project, this is fine. However, this can pose many challenges in a setting where multiple developers may be working on this project. First is that installing software is tedious, never fun and often prone to error. Second is that not all applications are availble on all operating systems. While postgres may be available on Windows, Mac and Linux, having that sort of cross-compatibility is not a given with a lot of software. Further, let's say a developer working on this project wanted to use postgres at home. Using the same installation of postgres for multiple projects is possible but messy. Beyond that, distributing what configurations are required to make the software work is difficult. Is that done on a readme? Sent as a message in Slack? Put in a text file? And what happens if those configurations update or change? How do you notify people? Finally, local installations are prone to being corrupted or messed up by other things running on the developer's machine.

It would be nice if there was a way to install postgres in such a way to avoid all these problems. And, since I'm bringing it up, you know there is. The answer to these problems is containerization. How does containerization work? Instead of running the app directly on our OS, we run another OS and run the app in that. By making the OS as small as possible, plus doing some other optimizations, this can be adequately fast and work well in a development environment. Even better, most containers (the OS and the app inside of it) can be configured via certain files. Thus, we can version control how an app like postgres is configured.

Since the container we wish to install and the settings for the app are controlled by configuration files, a developer doesn't have to worry about installing the software by hand. Most containzerization softwares are cross platform, too, meaning that you can run apps that usually aren't cross-platform inside of a container. Beyond that, you can have as many containers as you want in most containerization softwares, so you can have an installation of postgres for work and an installation of postgres for home if you so desire. Beyond that, developers no longer need to be notified when configurations change, the containerization software will just take the changed configurations and apply them. Finally, since our app is running in a completely separate OS, we don't have to worry about it being messed up or corrupted by other apps on our machine.

With all that in mind, let's use containers to make our postgres installation easier to work with. There are a variety of softwares that provide containerization, but we are going to use [Docker](https://www.docker.com/). A lot of people have started to dislike Docker in recent years for their pricing model, so, if you intend to use containerization in the future, you may want to consider other options, but, for our toy example purposes, Docker will work just fine. So, go to the site, create an account, and install Docker as a desktop app.

Now, from here, we'll be using the [Postgres - Official Image](https://hub.docker.com/_/postgres) to make our container. What's an image? It's just the OS with the app installed in it. The container is an instance running an image. So, we'll use an image to run a container that has postgres on it.

How are we going to be running this image? We'll be using the command-line tool [docker/compose](https://github.com/docker/compose), which is installed with the desktop installation of Docker. `docker-compose` takes a file called `docker-compose.yml` and uses that to determine what images to run and with what configurations.

By the way, as a note, `Dockerfile`s can be used to modify existing images, allowing you to build your own images that better fit your purposes. Luckily, we can get away with just using the base postgres image, but `Dockerfile`s are very often used in development environments, so I thought it should be mentioned.

In any case, we'll create a file called `docker-compose.yml` in the same directory as `Cargo.toml`. Inside of it will go the following code

```yml
version: '3.1'

services:

  db:
    image: postgres
    restart: always
    environment:
      POSTGRES_USER: "postgres"
      POSTGRES_PASSWORD: "password"
      POSTGRES_DB: "todo"
    ports:
    - "5432:5432"
```

Now, if you go to the `todo-app` directory, and run `docker-compose up`, your database will start. With the database running, you can then open up another terminal and run your app. However, there is still one problem. Since postgres is installed on your machine, it is constantly running a service that uses port 5432. Thus, when we access port 5432, we will be accessing our local app rather than the one in the container. Turn off or uninstall your postgres isntallation to allow you to connect to the containerized database. Once you do that, you'll notice nothing has changed, the database works the same, it's just easier to configure and work with now.

### Do We Need `docker-compose`?

No, we don't. We could run the image with the proper settings via the command line or the desktop app. But, using `docker-compose` allows us to leave a useful configuration file behind that details all the configurations used to make our app work. And, in the case we want multiple containers to be run for our app, we can list the configurations for all of them in `docker-compose.yml`.

I've never done this myself, but I'm pretty sure you can also use `Dockerfile`'s functionality to configure your image as well. So, if you want to get around using `docker-compose` for some weird reason, that may be an option. In any case, there's lots of ways to work with Docker, but I believe this one really maximizes the benefits you see from containerizing applications that work with your software.

# Proper Return Values

So far our returns have been pure data. Snippets of JSON, or strings. However, as I mentioned at the start, our app is supposed to take requests and return HTML, Javascript and CSS to display a webpage. Now, that's a bit of a lie. The `GET`, `PUT`, `POST` and `DELETE` methods we have already made are mainly going to stay the same. However, it is true that some of the paths in our server will return HTML. It just so happens that some paths return things that are being displayed, and some paths are used to do things. In any case, let's implement a path that actually returns something that is displayed.

We've been using Rocket pretty extensively to return values, as it is the last thing that touches our values and turns them into responses before they get sent to the browser. As can be imagined, Rocket also provides support for returning HTML, CSS and Javascript.

Now, we could put together a series of strings and make our own HTML file with CSS and Javascript simply by manipulating strings, but I'm not going to do that. It's tedious, it's long, and I don't think it really provides anything of value. We're going to be skipping straight to the abstraction built on top of that, which is to use a templating engine. Basically, we write out an HTML file, but we leave places where data can be entered, and we provide that data before sending the template back. It's obvious why this abstraction exists. It's easier to write a fancy HTML file and provide the necessary data, rather than dealing with a whole spaghetti'd mess of strings.

As expected, Rocket allows you to use and return Templates, and it also allows you to choose which template engine you want to use from a list (a template engine is the thing that takes our template and turns it into proper HTML). For our purposes we are going to be using the templating engine called [[Tera](https://tera.netlify.app/)](https://handlebarsjs.com/).

To start, we need to configure Rocket to pull our templates from a certain directory, so modify `Rocket.toml` to look like the following

```toml
[default]
template_dir = "templates/"

[default.databases.todo]
url = "postgres://postgres:password@localhost:5432/todo"
```

After that, we need to import the Rocket library that will interpret the template, so modify `Cargo.toml` to look like the following

```toml
[package]
name = "todo-app"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[workspace]
members = [".", "entity", "migration"]

[dependencies]
async-trait = { version = "0.1" }
entity = { path = "entity" }
migration = { path = "migration" }

[dependencies.rocket]
version = "0.5.0-rc.2"
features = ["json"]

[dependencies.rocket_dyn_templates]
version = "0.1.0-rc.1"
features = ["tera"]

[dependencies.sea-orm-rocket]
git = "https://github.com/SeaQL/sea-orm"

[dependencies.sea-orm]
version = "^0.8.0"
features = [
  "runtime-tokio-native-tls",
  "sqlx-postgres",
  "macros"
]
```

Now, at the same level as that `Rocket.toml`, create a directory called `templates`. For the templates we create, we are going to use the file extension `.tera` at the end to let Rocket know that we want to compile this with handlebars. Speaking of, in the `templates` directory, create a file called `base.html.tera`. This file is basically going to serve as the wrapper for all our content. Every other template will basically say, "start in the middle of `base.html.tera`", so we only have to write out `<!DOCTYPE html>` and all those other tedious imports once. Now, add the following code to `base.html.tera`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Todo App</title>
    <meta name="description" content="A todo app created via Rocket and Rust" />
    <meta name="author" content="Garrett Udstrand" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />

  </head>
  <body>
    <div class="container">
      {% block content %}{% endblock content %}
    </div>
  </body>
</html>
```

Now we are going to create the actual list of todo items. Create a file called `todo_list.html.tera`. Fill it with the following code (which will place content in the block we named content)

```html
{% extends "base" %} {% block content %}
<h1>Todo List</h1>
<table>
    <colgroup>
        <col class="medium">
        <col class="wide">
        <col class="medium">
    </colgroup>
    <tr>
        <th>ID</th>
        <th>Item</th>
        <th>Actions</th>
    </tr>
    {% for task in tasks %}
    <tr>
        <td>{{ task.id }}</td>
        <td class="item">{{ task.item }}</td>
        <td >
            <div class="actions">
                <button>Edit</button>
                <form action="/deletetask/{{ task.id }}" method="post">
                    <input type="hidden" name="_method" value="delete" />
                    <button type="submit">Delete</button>bmit">Delete</button>
                </form>
            </div>
        </td>
    </tr>
    {% endfor %}
</table>
<form action="/addtask" method="post">
    <div class="add_task_form">
        <input type="text" name="item" placeholder="Enter a new task here..." />
        <button type="submit">Add Task</button>
    </div>
</form>
{% endblock content %}
```

As you can see, we're using the templating engine to dynamically create items based on the data that we will be pulling from the database. Over in `main.rs`, we'll set up the function to render our template properly and add our template engine to the Rocket app. The code for it looks like this

```rust
#[get("/")]
async fn index(conn: Connection<'_, Db>) -> Result<Template, DatabaseError> {
    let db = conn.into_inner();
    let tasks = Tasks::find()
                            .order_by_asc(tasks::Column::Id)
                            .all(db)
                            .await?;

    Ok(Template::render(
        "todo_list",
        json!({
            "tasks": tasks
        })
    ))
}

#[launch]
fn rocket() -> _ {
    rocket::build()
        .attach(Db::init())
        .attach(AdHoc::try_on_ignite("Migrations", run_migrations))
        .mount("/", routes![index, add_task, read_tasks, edit_task, delete_task])
        .attach(Template::fairing())
}
```

With that, if we run the program again, and open up the link in our browser, we'll see a list of the items from our database. However, it doesn't look particularly good. It would be nice if we could use some CSS to improve the looks, but how do we get CSS to our frontend? This takes us down the path of serving static content.

## Serving Static Content

Static content is content that does not change. This means that, regardless of what the user does, regardless of what happens in the application, the file will not change. Most CSS files and images are static content. They don't really change over the course of the application. They just exist and provide additional information.

Often times, we want static content to be freely available. In other words, we'd really like to be able to just put in a quick `GET` request and pull back this content without much fuss. Luckily, most web frameworks, including Rocket, provide a functionality to expose a folder filled with static content. After this folder is "exposed", we can run `GET` requests like we're moving through paths in that folder and can pull back the associated files.

So, let's place any and all static content in a new folder called `public` that sits at the same level as `Cargo.toml`. In this folder, we'll put all the CSS, images and other static content we want our site to be able to use. Then, we'l use Rocket to expose that folder. The code for making Rocket expose that folder happens in `main.rs` and looks like the following

```rust
#[launch]
fn rocket() -> _ {
    rocket::build()
        .attach(Db::init())
        .attach(AdHoc::try_on_ignite("Migrations", run_migrations))
        .mount("/", FileServer::from(relative!("/public")))
        .mount("/", routes![index, add_task, read_tasks, edit_task, delete_task])
        .attach(Template::fairing())
}
```

As you can see, we've mounted all the files inside of public to be used from `/`. That means, if we have a file called `todo_list.css` in our public folder, calling `http://127.0.0.1:8000/todo_list.css` will return that CSS. If we have a folder in public called `images` and we put an image called `pizza.png` in there, calling `http://127.0.0.1:8000/images/pizza.png` will return that image. Using this, we can now write CSS for our templates, and can retreive them when we are on the frontend.

So, let's use our new ability to serve static content! In the `public` folder, create a file called `todo_list.css` and put in this code

```css
html {
    font-family: sans-serif;
}

table {
    border-collapse: collapse;
    border: 2px solid rgb(200,200,200);
    letter-spacing: 1px;
    font-size: 0.8rem;
    width:100%;
    margin-bottom: 2rem;
}

td, th {
    border: 1px solid rgb(190,190,190);
    padding: 10px 20px;
}

th {
    background-color: rgb(235,235,235);
}

td {
    text-align: center;
}

tr:nth-child(even) td {
    background-color: rgb(250,250,250);
}

tr:nth-child(odd) td {
    background-color: rgb(245,245,245);
}

caption {
    padding: 10px;
}

.medium {
    width: 20%;
}

.wide {
    width: 60%;
}

.item {
    text-align: left;
}

button {
    padding: 10px 20px;
    background: transparent;
    border-radius: 0.25rem;
    border-color: #ddd;
    border-width: 1px;
    cursor: pointer;
    border-style: solid;
    transition: background-color 0.2s ease-in;
    letter-spacing: 1px;
    font-size: 0.8rem;
}

button:hover {
    background-color: #ddd;
}

input {
    border-radius: 0.25rem;
    border-color: #ddd;
    border-width: 1px;
    padding: 10px 20px;
    border-style: solid;
    width: 50%;
    letter-spacing: 1px;
    font-size: 0.8rem;
}

.add_task_form {
    display: flex;
    justify-content: space-between;
}

.actions {
    display: flex;
    justify-content: space-around;
}
```

Now, in our `base.html.tera` file we need to import the CSS. To do that, we need to use a `link` element to pull in a stylesheet. We use the `href` prop to pass in a URL that contains CSS. If now start to the URL is given, it is assumed the base URL is our website.

In non jargon-y terms, put the following code beneath the curly bracket parts at the top of `todo_list.html.tera`

```html
<link rel="stylesheet" href="/todo_list.css" />
```

Now, if you re-run your app, the table should look quite a bit better. Now, in the spirit of doing a little bit of styling, let's create another file in `public` called `container.css` with the following code

```css
.container {
    max-width: 1200px;
    margin: auto;
    padding-top: 2rem;
}
```

And add the following stylesheet link between the meta tags in `base.html.tera`

```html
<link rel="stylesheet" href="/container.css" />
```

With that, our styling looks a bit more passable.

## The Code To Fix Our Responses

Now, our functions are still returning improper values. Even after implementing the previous fix, if you try and add a task, you will be left with an empty page filled with JSON, which we don't want. What we'd rather do is re-direct to our TODO list page and add a little notification saying something like "Task created!". Luckily, Rocket has a structure called `Flash` that will do that for us perfectly.

So, with that, let's modify our `add_task` function to work more properly with our app! All we need to do is modify it to use that `Flash` structure to redirect and send a message

```rust
#[post("/addtask", data="<task_form>")]
async fn add_task(conn: Connection<'_, Db>, task_form: Form<tasks::Model>) -> Result<Flash<Redirect>, DatabaseError> {
    let db = conn.into_inner();
    let task = task_form.into_inner();

    let active_task: tasks::ActiveModel = tasks::ActiveModel {
        item: Set(task.item),
        ..Default::default()
    };

    active_task.insert(db).await?;

    Ok(Flash::success(Redirect::to("/"), "Task created!"))
}
```

Next, we modify `index` to recieve that message and pass it along to our HTML

```rust
#[get("/")]
async fn index(conn: Connection<'_, Db>, flash: Option<FlashMessage<'_>>) -> Result<Template, DatabaseError> {
    let db = conn.into_inner();
    let tasks = Tasks::find()
                            .order_by_asc(tasks::Column::Id)
                            .all(db)
                            .await?;

    Ok(Template::render(
        "todo_list",
        json!({
            "tasks": tasks,
            "flash": flash.map(FlashMessage::into_inner)
        })
    ))
}
```

Then, we modify the top of `todo_list.html.tera` to use that message.

```html
{% extends "base" %} {% block content %}
<h1>Todo List</h1>
{% if flash %}
<p class="{{ flash.0 }}-flash">
  {{ flash.1 }}
</p>
{% endif %}
...
```

Finally, add the following CSS to `container.css`

```css
.error-flash {
    margin: -10px 0 10px 0;
    color: red;
}

.success-flash {
    margin: -10px 0 10px 0;
    color: green;
}
```

Let's now modify the delete task to also work properly. Change the function to look like this

```rust
#[delete("/deletetask/<id>")]
async fn delete_task(conn: Connection<'_, Db>, id: i32) -> Result<Flash<Redirect>, DatabaseError> {
    let db = conn.into_inner();
    let _result = Tasks::delete_by_id(id).exec(db).await?;

    Ok(Flash::success(Redirect::to("/"), "Task succesfully deleted!"))
}
```

With that, our deletes should now work properly.

### The Difficult Edit Operation

Sadly, the edit operation is going to make our lives more difficult. The easiest way to implement it is going to be to create another route in our application that opens a new page. In this page, we can then edit the task and submit that edit.

First, we need to create the template that we are going to be visiting. Create a new file in `templates` called `edit_task_form.html.tera` and enter the following code into it

```html
{% extends "base" %} {% block content %}
<form action="/" method="get" class="back-button">
    <button type="submit">Back</button>
</form>
<h1>Task - {{ task.id }}</h1>
<form action="/edittask" method="post">
    <input type="hidden" name="_method" value="put" />
    <input type="hidden" name="id" value="{{ task.id }}" />

    <label for="item">Item:</label>
    <div class="item-input">
        <input type="text" name="item" value="{{ task.item }}" />
    </div>

    <button type="submit">Save</button>
</form>
{% endblock content %}
```

Then, in `main.rs`, add the following code

```rust
#[get("/edit/<id>")]
async fn edit_task_page(conn: Connection<'_, Db>, id: i32) -> Result<Template, DatabaseError> {
    let db = conn.into_inner();
    let task = Tasks::find_by_id(id).one(db).await?.unwrap();

    Ok(Template::render(
        "edit_task_form", 
        json!({
            "task": task
        })
    ))
}
```

And, of course, put that in the `routes!` macro in `rocket`. Next, modify the edit button in `todo_list.html.tera` to look like this instead

```html
<form action="/edit/{{ task.id }}" method="get">
    <button type="submit">Edit</button>
</form>
```

After that, go into `main.rs` and modify the `edit_task` function like follows

```rust
#[put("/edittask", data="<task_form>")]
async fn edit_task(conn: Connection<'_, Db>, task_form: Form<tasks::Model>) -> Result<Flash<Redirect>, DatabaseError> {
    let db = conn.into_inner();
    let task = task_form.into_inner();

    let task_to_update = Tasks::find_by_id(task.id).one(db).await?;
    let mut task_to_update: tasks::ActiveModel = task_to_update.unwrap().into();
    task_to_update.item = Set(task.item);
    task_to_update.update(db).await?;

    Ok(Flash::success(Redirect::to("/"), "Task edited succesfully!"))
}
```

Finally, go into `todo_list.css` and add the following CSS

```css
label {
    display: block;
    letter-spacing: 1px;
    font-size: 1rem;
    font-weight: bold;
    margin-bottom: 0.3rem;
}

.item-input {
    margin-bottom: 1.5rem;
}

.back-button {
    margin-bottom: 4rem;
}
```

With that, if you stop and restart the app, and try out the web part. You can now properly use all the CRUD operations.

## Dealing With Errors

Right now, the way we deal with errors is inadequate. Even though our errors don't crash the server, they still take a user to a 404, 500 or whatever status page. It would be much better if the error was better integrated into the app and didn't chop up the user experience as much. We can use the same `Flash` struct we have been using for our succesful returns, and also use it for our bad returns. In fact, `Flash` supports this with the associated function `Flash::error`. So, we're just going to go through our functions that don't return templates and have them return `Flash::error`s when they fail, rather than sending a user to a 404 or 500 page. Here's `delete_task`'s code

```rust
#[delete("/deletetask/<id>")]
async fn delete_task(conn: Connection<'_, Db>, id: i32) -> Flash<Redirect> {
    let db = conn.into_inner();
    let _result = match Tasks::delete_by_id(id).exec(db).await {
        Ok(value) => value,
        Err(_) => {
            return Flash::error(Redirect::to("/"), "Issue deleting the task");
        }
    };

    Flash::success(Redirect::to("/"), "Task succesfully deleted!")
}
```

Here's `edit_task`'s code

```rust
#[put("/edittask", data="<task_form>")]
async fn edit_task(conn: Connection<'_, Db>, task_form: Form<tasks::Model>) -> Flash<Redirect>{
    let db = conn.into_inner();
    let task = task_form.into_inner();

    let task_to_update = match Tasks::find_by_id(task.id).one(db).await {
        Ok(result) => result,
        Err(_) => {
            return Flash::error(Redirect::to("/"), "Issue editing the task");
        }
    };
    let mut task_to_update: tasks::ActiveModel = task_to_update.unwrap().into();
    task_to_update.item = Set(task.item);
    match task_to_update.update(db).await {
        Ok(result) => result,
        Err(_) => {
            return Flash::error(Redirect::to("/"), "Issue editing the task");
        }
    };

    Flash::success(Redirect::to("/"), "Task edited succesfully!")
}
```

Here's `add_task`

```rust
#[post("/addtask", data="<task_form>")]
async fn add_task(conn: Connection<'_, Db>, task_form: Form<tasks::Model>) -> Flash<Redirect> {
    let db = conn.into_inner();
    let task = task_form.into_inner();

    let active_task: tasks::ActiveModel = tasks::ActiveModel {
        item: Set(task.item),
        ..Default::default()
    };

    match active_task.insert(db).await {
        Ok(result) => result,
        Err(_) => {
            return Flash::error(Redirect::to("/"), "Issue creating the task");
        }
    };

    Flash::success(Redirect::to("/"), "Task created!")
}
```

And, you may have noticed that `read_tasks` no longer really needs to be used, as `index` basically does the same thing now, so feel free to either keep that function for debugging, or remove it if you don't want it.

While you may not be able to see an error unless you go out of your way to induce one, just realize that, when errors do happen, it does work and looks something like this

![](file://C:\Users\garre\AppData\Roaming\marktext\images\2022-06-24-17-53-50-image.png?msec=1656373992261)

Now, you may have noticed that I did not switch up the `DatabaseError`'s that come back from the routes that return `Template`s. That's because if the page fails to load, it's much more normal for a 404 or 500 page to come up to take its place. So, it works there.

## Pagination

Now, our Todo List is a list of an unknown number of items. A person could have 100s of tasks in here, or they could have 3. Generally, when we display a list of data, and that list can potentially be very long, we have to paginate our data. What that means is we split our list into pages, and each of these pages has a certain number of items in the list on it. Then, when we want to view items, we only show one page of items at a time.

`<Quick Aside>`

In many social media apps like Reddit, Instagram, Twitter or Facebook, the home page will be a list of posts from people you are following. However, you can keep on scrolling for as long as you want. When you reach the end of the posts on the home page, it just loads more and you can then view those. We often call this an infinitely scrollable lists. However, this is also pagination! The app is obviously not just loading an infinite number of posts. It also loads posts in chunks or pages and puts them on the list, it just tries to get the next page of posts and add them to the list just before you reach the bottom of it.

`</Quick Aside>`

Obviously, we want to implement this for our app. Not only to learn about pagination, but because it will legitimately make this app easier to use.

To do this, however, we will need to change up how `index` works a little bit. Instead of just returning every task we have in our database, we'll instead want to take a parameter to know what page of tasks the user wants. Further, we'll want the browser to be able to specify how many tasks each page has. That could be used later to make the list work better for different devices by changing the number of tasks per page based on its vertical size. We could also just put an input up and allow users to specify how many items they want on each page. Both could work well, but that is clearly an option we'll want.

To add it is simple enough. Pagination is provided by `sea-orm`. Just modify `index` to look like the following

```rust
#[get("/?<page>&<tasks_per_page>")]
async fn index(conn: Connection<'_, Db>, flash: Option<FlashMessage<'_>>, page: Option<usize>, tasks_per_page: Option<usize>) -> Result<Template, DatabaseError> {
    let db = conn.into_inner();
    let page = page.unwrap_or(0);
    let tasks_per_page = tasks_per_page.unwrap_or(5);

    let paginator = Tasks::find()
                            .order_by_asc(tasks::Column::Id)
                            .paginate(db, tasks_per_page);
    let number_of_pages = paginator.num_pages().await?;
    let tasks = paginator.fetch_page(page).await?;


    Ok(Template::render(
        "todo_list",
        json!({
            "tasks": tasks,
            "flash": flash.map(FlashMessage::into_inner),
            "number_of_pages": number_of_pages,
            "current_page": page
        })
    ))
}
```

You may notice that the route of `index` now has some new parameters. The way they are written makes them optional, so they are not required to open the page. In any case, modify `todo_list.html.tera` to look like the following

```html
{% extends "base" %} {% block content %}
<h1>Todo List</h1>
{% if flash %}
<p class="{{ flash.0 }}-flash">
  {{ flash.1 }}
</p>
{% endif %}
<table>
    <colgroup>
        <col class="medium">
        <col class="wide">
        <col class="medium">
    </colgroup>
    <tr>
        <th>ID</th>
        <th>Item</th>
        <th>Actions</th>
    </tr>
    {% for task in tasks %}
    <tr>
        <td>{{ task.id }}</td>
        <td class="item">{{ task.item }}</td>
        <td >
            <div class="actions">
                <form action="/edit/{{ task.id }}" method="get">
                    <button type="submit">Edit</button>
                </form>
                <form action="/deletetask/{{ task.id }}" method="post">
                    <input type="hidden" name="_method" value="delete" />
                    <button type="submit">Delete</button>
                </form>
            </div>
        </td>
    </tr>
    {% endfor %}
</table>
<div class="page-row">
    {% if current_page == 0 %}
        {% set start_page = current_page + 1 %}
    {% else %}
        {% set start_page = current_page %}
    {% endif %}

    {% if start_page + 3 > number_of_pages %}
        {% set end_page = number_of_pages + 1 %}
    {% else %}
        {% set end_page = start_page + 3 %}
    {% endif %}

    {% for i in range(start = start_page, end = end_page) %}
        <a href="/?page={{ i - 1 }}" class="page-selector">{{ i }}<a>
    {% endfor %}

    {% if end_page <= number_of_pages %}
        <a>...</a>
    {% endif %}
</div>
<form action="/addtask" method="post">
    <div class="add_task_form">
        <input type="text" name="item" placeholder="Enter a new task here..." />
        <button type="submit">Add Task</button>
    </div>
</form>
{% endblock content %}
```

Finally, modify `todo_list.css` to look like the following

```css
html {
    font-family: sans-serif;
}

table {
    border-collapse: collapse;
    border: 2px solid rgb(200,200,200);
    letter-spacing: 1px;
    font-size: 0.8rem;
    width:100%;

}

td, th {
    border: 1px solid rgb(190,190,190);
    padding: 10px 20px;
}

th {
    background-color: rgb(235,235,235);
}

td {
    text-align: center;
}

tr:nth-child(even) td {
    background-color: rgb(250,250,250);
}

tr:nth-child(odd) td {
    background-color: rgb(245,245,245);
}

caption {
    padding: 10px;
}

.medium {
    width: 20%;
}

.wide {
    width: 60%;
}

.item {
    text-align: left;
}

button {
    padding: 10px 20px;
    background: transparent;
    border-radius: 0.25rem;
    border-color: #ddd;
    border-width: 1px;
    cursor: pointer;
    border-style: solid;
    transition: background-color 0.2s ease-in;
    letter-spacing: 1px;
    font-size: 0.8rem;
}

button:hover {
    background-color: #ddd;
}

input {
    border-radius: 0.25rem;
    border-color: #ddd;
    border-width: 1px;
    padding: 10px 20px;
    border-style: solid;
    width: 50%;
    letter-spacing: 1px;
    font-size: 0.8rem;
}

.add_task_form {
    display: flex;
    justify-content: space-between;
}

.actions {
    display: flex;
    justify-content: space-around;
}

label {
    display: block;
    letter-spacing: 1px;
    font-size: 1rem;
    font-weight: bold;
    margin-bottom: 0.3rem;
}

.item-input {
    margin-bottom: 1.5rem;
}

.back-button {
    margin-bottom: 4rem;
}

.page-row {
    margin-bottom: 2rem;
    margin-top: 1rem;
}

.page-selector {
    padding: 1rem 1rem;
}
```

With that, we have no added basic pagination. There is a usability problem with this pagination, however. Since our add, delete, and edit tasks all go to the route `/`, whenever we try to do new things to our list, we go back to the first page, which is not ideal. However, it works for our purposes of demonstrating pagination. We could also add in other great usability features like filtering and sorting our list, but I won't be doing that here. I don't think they're really as important to the greater architecture of web apps. If you want to know, however, `sql` and thus SeaORM implement ways to filter and sort items, and you just use those on whatever query you run to get your items back.

# Finally Addressing the Multiple User Issue

As I mentioned WAY back, our app has a problem where it only properly works for one user. Or, at the very least, everyone sees everyone else's tasks that they add. Obviously, we don't want that. We want each user who uses the app to see their own content. In any app where a user adds their own data, or edits their own set of data, we have to implement some sort of system to account for multiple users.

Since accounting for multiple users is a quite common problem, most frameworks have some built-in way to deal with it, and, even if yours doesn't, there are a few common patterns for handling multiple users. Rocket doesn't have anything that specifically authenticates users, but it does have a feature we've been using called Request Guards, which allows us to validate or authenticate data before it comes in. We'll be able to use that to ensure that a user is signed in before accessing or using certain routes.

To start, though, we need to create a users table in our database. Why? Because we'll need to keep track of what users are using our app, and we need to keep track of there password, so that, when they try to log in, we can verify that the person is who they say they are. And, whenever we need to hold onto or keep track of data in a web app, the first thought should be to create a table in the database for it.

You may wonder if storing a password in our database is secure or safe. The short answer is yes. The long answer is that it depends on how you store the password. Many people do this wrong, so it can be insecure. [This article](https://blog.codinghorror.com/youre-probably-storing-passwords-incorrectly/) talks a lot about the problem of storing passwords. There's also a greater argument to be made about whether every site should be implementing their own authentication. Maybe we should only have one authentication style for everything.

However, this is not an easy thing to implement, and whether we even should be trying to do something like that is not universally agreed upon. Just look at the discussion in [this article](https://blog.codinghorror.com/the-login-explosion/) and its comments. However, that article was written in 2006, and things like [LastPass](https://www.lastpass.com/) or [1Password](https://1password.com/) basically solve this problem of login explosion. In any case, the truth is that many web apps have their own authentication, so it's useful to learn some of the basics of how it often works. With that settled, let's go to our terminal and run

```bash
sea-orm-cli migrate generate create_users_table
```

This will create a new migration that will be creating the table we store users in once we are done. But we first need to write some code to make that happen, so go to the newly created file in `migration/src`. We're going to write code that will make a table called `users` with an id and two text columns: one to store the username and one to store the password. The code looks like this

```rust
use sea_orm_migration::prelude::*;

pub struct Migration;

impl MigrationName for Migration {
    fn name(&self) -> &str {
        "m20220624_213213_create_users_table"
    }
}

#[async_trait::async_trait]
impl MigrationTrait for Migration {
    async fn up(&self, manager: &SchemaManager) -> Result<(), DbErr> {
        manager
            .create_table(
                sea_query::Table::create()
                    .table(Users::Table)
                    .if_not_exists()
                    .col(
                        ColumnDef::new(Users::Id)
                            .integer()
                            .not_null()
                            .auto_increment()
                            .primary_key(),
                    )
                    .col(ColumnDef::new(Users::Username).string().unique_key().not_null())
                    .col(ColumnDef::new(Users::Password).string().not_null())
                    .to_owned()
            )
            .await
    }

    async fn down(&self, manager: &SchemaManager) -> Result<(), DbErr> {
        manager
            .drop_table(
                sea_query::Table::drop()
                    .table(Users::Table)
                    .to_owned()
            )
            .await
    }
}

#[derive(Iden)]
pub enum Users {
    Table,
    Id,
    Username,
    Password
}
```

With that completed, let's run `cargo run` to call this migration. After that, go to `entity/src` and create a file called `users.rs`. Then, enter the following

```rust
use rocket::{serde::{Deserialize, Serialize}, FromForm};
use sea_orm::entity::prelude::*;

#[derive(Clone, Debug, PartialEq, DeriveEntityModel, Deserialize, Serialize, FromForm)]
#[serde(crate = "rocket::serde")]
#[sea_orm(table_name = "users")]
pub struct Model {
    #[sea_orm(primary_key)]
    #[field(default = 0)]
    pub id: i32,
    pub username: String,
    pub password: String
}

#[derive(Copy, Clone, Debug, EnumIter)]
pub enum Relation {}

impl RelationTrait for Relation {
    fn def(&self) -> RelationDef {
        panic!("No RelationDef")
    }
}

impl ActiveModelBehavior for ActiveModel {}
```

Finally, go into `entity/src/lib.rs` and make it the following code

```rust
pub mod tasks;
pub mod users;
```

With that, the table, the migration and the entity for our users is all done. Now, we can work on implementing all the features we want related to users. As you can see, setting up the ORM was quite a pain, but now that we have it, it saves us a bunch of time in adding new tables and features. You can see how, if you have an app that may end up with 100s of tables, this could be quite the boon.

In any case, we need to write a few thing for our users. We need to have one request that returns a log in page, one request that returns a sign up page, a request that creates a user and a request that logs in a user. Then, of course, after we've done all that, we need to use that Request Guard stuff to actually authenticate people, and we need to rewrite our current requests to only allow us to create, edit or delete stuff for specific users.

That's a lot, but we'll go through it step-by-step, and we'll see that each step in the process is actually not that large.

## Signup Page

First, let's create the frontend for the signup page. Create a file called `authorization.css` and add the following code

```css
.authorization-box  {
    max-width: 500px;
    margin: auto;
    border: 1px solid #ddd;
    border-radius: 0.25rem;
    padding: 2rem;
}

input {
    border-radius: 0.25rem;
    border-color: #ddd;
    border-width: 1px;
    padding: 10px 20px;
    border-style: solid;
    width: 100%;
    letter-spacing: 1px;
    font-size: 0.8rem;
    margin-bottom: 1rem;
}

label {
    display: block;
    letter-spacing: 1px;
    font-size: 1rem;
    font-weight: bold;
    margin-bottom: 0.3rem;
}

button {
    padding: 10px 20px;
    background: rgb(165, 0, 165);
    border-radius: 0.25rem;
    cursor: pointer;
    transition: background-color 0.2s ease-in;
    letter-spacing: 1px;
    font-size: 1rem;
    font-weight: bold;
    border: none;
    color: white;
}

button:hover {
    background-color: purple;
}
```

Next, over in `container.css`, add the following code

```css
* {
    box-sizing: border-box;
}
```

Finally, create a file called `signup_page.html.tera` in `templates` and add the following code

```html
{% extends "base" %} {% block content %}
<link rel="stylesheet" href="/authorization.css" />
<form action="/createaccount" method="post">
    <div class="authorization-box">
        {% if flash %}
            <p class="{{ flash.0 }}-flash">
                {{ flash.1 }}
            </p>
        {% endif %}

        <label for="username">Username:</label>
        <input type="text" id="username" name="username" />

        <label for="password">Password:</label>
        <input type="password" id="password" name="password" />

        <button type="submit">Sign Up</button>
    </div>
</form>
{% endblock content %}
```

Now, in `main.rs`, we add the route to serve the template

```rust
#[get("/signup")]
async fn signup_page(flash: Option<FlashMessage<'_>>) -> Template {
    Template::render(
        "signup_page", 
        json!({
            "flash": flash.map(FlashMessage::into_inner)
        })
    )
}
```

Of course, put `signup_page` in the routes.

## Create Account Request

The signup page relies on a post method that has `createaccount` as the route. So, we have to discuss how we're going to store our users. For the most part, it will be just like storing tasks. They go into a table, and, when we need some information from that table, we read from it. However, we need to be more secure with how we store user's passwords, because if someone gains access to our database, we don't want them to be able to easily sign in to other people's accounts. We also don't want these bad actors to determine what someone's password is at all, because people often use the same password for multiple websites.

Thus, when we store a user's password, we don't just store it in plain text. Instead, we use a hashing or encryption function to turn the password into a string of much less readable characters. The hope is that the hashing function is not reversable, so, someone with a hash cannot determine what has input to create that hash. This means that the user's password is safe for the most part, even if a bad actor gains access to our database.

How do we determine the right password is input if we have a random string of characters? We simply hash the input password and see if the hash of the input password matches the hash we have on hand. Thus, everything is kept secure, but people are still able to create and sign into accounts.

There is A LOT more that goes into encryption and hashing and all the details around that, but, for our purposes, we just need to know encryption functions exist and how we use them. For our project, we are going to be using a password-hashing function known as [Argon2](https://github.com/P-H-C/phc-winner-argon2). Specifically we'll be using [this implementation](https://docs.rs/rust-argon2/latest/argon2/) created for Rust. So, in the root `Cargo.toml`, put the following line under dependencies

```rust
rust-argon2 = "1.0"
```

Now, at the top of `main.rs`, put the following line of code

```rust
extern crate argon2;
```

Over in the `users` entity, add the following line of code

```rust
pub const USER_PASSWORD_SALT: &[u8] = b"some_random_salt";
```

We need to have what is known as a "salt" for our hashing function to work. Ideally, this salt is kept secure and not put in a place where it will go into a public repository. So, know that me putting the salt here in the entity is not a great idea for a real project, but getting into the nuances of storing certain pieces of data securely for our app is a bit too much for this discussion now (if you're curious how we'd keep this secret, we'd just put this in a file that doesn't get put on GitHub. Then, for any new developer joining the project, you'd share the salt with them over some secure messaging app. Of course, this isn't the only way of keeping it secret/secure, and other, better methods exist, too).

Finally, add the following function to `main.rs`

```rust
#[post("/createaccount", data="<user_form>")]
async fn create_account(conn: Connection<'_, Db>, user_form: Form<users::Model>) -> Flash<Redirect> {
    let db = conn.into_inner();
    let user = user_form.into_inner();

    let hash_config = Config::default();
    let hash = match argon2::hash_encoded(user.password.as_bytes(), USER_PASSWORD_SALT, &hash_config) {
        Ok(result) => result,
        Err(_) => {
            return Flash::error(Redirect::to("/signup"), "Issue creating account");
        }
    };

    let active_user = users::ActiveModel {
        username: Set(user.username),
        password: Set(hash),
        ..Default::default()
    };

    match active_user.insert(db).await {
        Ok(result) => result,
        Err(_) => {
            return Flash::error(Redirect::to("/signup"), "Issue creating account");
        }
    };

    Flash::success(Redirect::to("/login"), "Account created succesfully!")
}
```

This takes in the username and password given by the user and uses our hashing function to store the new user. Of course, we have flashes and redirects to handle successes and errors. It should look a lot like `add_task`, because both are creation operations. This one just has a couple extra bells and whistles.

Finaly, mount this route in `rocket` like we've been doing for everything else. With that, we've made the signup page and it's associated action.

## Login Page

Next is the login page! We're going to jack most of the look and feel of the sign up page, so we don't even need to create a new CSS file. Just make `login_page.html.tera` in `templates` and enter the following code

```html
{% extends "base" %} {% block content %}
<link rel="stylesheet" href="/authorization.css" />
<form action="/verifyaccount" method="post">
    <div class="authorization-box">
        {% if flash %}
            <p class="{{ flash.0 }}-flash">
                {{ flash.1 }}
            </p>
        {% endif %}

        <label for="username">Username:</label>
        <input type="text" id="username" name="username" />

        <label for="password">Password:</label>
        <input type="password" id="password" name="password" />

        <button type="submit">Login</button>
        <p>Don't have an account? <a href="/signup">Signup here!</a></p>
    </div>
</form>
{% endblock content %}
```

Now, in `main.rs`, let's create the route that will display this page.

```rust
#[get("/login")]
async fn login_page(flash: Option<FlashMessage<'_>>) -> Template {
    Template::render(
        "login_page", 
        json!({
            "flash": flash.map(FlashMessage::into_inner)
        })
    )
}
```

As you may have noticed, the login and signup page are very similar, and we may be able to package up their similar functionality into functions or templates or whatever else. However, whether you do that is dependant on how much you want the two to actually look the same. A lot of the code in this example is not perfect or amazingly clean, but it gets the job done. But, if you wish to clean up the code into something prettier, go ahead! It may be a good project to really think through everything being done.

In any case, our login page is now complete, so now we can implement the route that is going to verify our account

## Verify Account Route

Now, the verifying of the account shouldn't be too bad. We use the `username` given to us to find the user in the database, and verify that the password given matches the password in the database. Without any helper functions, we would just hash the password given and then compare the given password to the stored password. However, the Argon2 library provides a method for verifying passwords, so we'll be using that. In the case the password is not correct, we will use a flash to tell that to the user. If the password is correct, we set a cookie with the authenticated user's id and use a flash to tell the user they have logged in succesfully.

```rust
fn set_user_id_cookie(cookies: & CookieJar, user_id: i32) {
    cookies.add_private(Cookie::new("user_id", user_id.to_string()));
}

#[post("/verifyaccount", data="<user_form>")]
async fn verify_account(conn: Connection<'_, Db>, cookies: & CookieJar<'_>, user_form: Form<users::Model>) -> Flash<Redirect> {
    let db = conn.into_inner();
    let user = user_form.into_inner();

    let stored_user = match Users::find()
        .filter(users::Column::Username.contains(&user.username))
        .one(db)
        .await {
            Ok(model_or_null) => {
                match model_or_null {
                    Some(model) => model,
                    None => {
                        return login_error();
                    }
                }
            },
            Err(_) => {
                return login_error();
            }
        };
    
    let is_password_correct = match argon2::verify_encoded(&stored_user.password, user.password.as_bytes()) {
        Ok(result) => result,
        Err(_) => {
            return Flash::error(Redirect::to("/login"), "Encountered an issue processing your account")
        }
    };

    if !is_password_correct {
        return login_error();
    }

    set_user_id_cookie(cookies, stored_user.id);
    Flash::success(Redirect::to("/"), "Logged in succesfully!")
}
```

As you can see, most of our code goes into finding the user and returning appropriate errors when things go wrong. Only the bottom is really concerned with verifying the password. This is pretty messy code, and we'll do *a little* work on cleaning this up once everything is finished up, but we still have some more authentication things to do

## Using Our Account Verification

Now that we have the cookie set and our account verified, we are going to create a request guard to ensure our account is verified when entering certain routes. A request guard, in terms of Rocket, just allows us to perform some arbitrary verification on whataver request we attach it to. Here, we're going to verify that a `user_id` cookie is attached to the request.

This is not perfect. Ideally, the data stored in our cookie would tell us a lot more than the `id` of the user verified. It would also tell us when the user signed in, how long the cookie is valid for and other details. Having that information on hand could allow us to make decisions to make our site more secure. Beyond that, if the cookie we set is more complicated than just a simple id, it will be harder to forge by bad actors. [JSON Web Tokens](https://jwt.io/), or JWT, do something exactly like that. JWT works by taking a JSON object and encoding that into a string. Then, if a JWT were to come into the backend, we would decode the JWT back into JSON. That way, we can have all the information I just mentioned and anything else we want. Even better, this JWT can be signed to make it easier to trust the validity of it and encrypted to make it harder for bad actors to reverse engineer. Thus, in an actual web application, you would probably use something like JWT for an authorization cookie, rather than just using an ID.

Why am *I* not using something like JWT for this example? Mainly because it requires me to do a lot more work, the authorization is already a big hassle to add to this project, and I've nearly written 19000 words about this web app and am starting to get quite fed up with adding extra features, especially since we're so close to the end.

In any case, this is a toy example, so the code is a bit messy and the choices I make are not really great for security. The more important point to take away is a lot of the choices I am making emulate how a lot of web apps are designed. While no competent web app would just set the user id as a cookie and use that for verification, the idea of a cookie being used to verify incoming requests is used quite a bit. It's just that, instead of an id, the cookie will usually be holding something like a JWT.

Regardless, let's code the request guard. The first task is to import a libary we should've imported much earlier called `anyhow`. This is a library to make error handling in Rust a lot easier. Note, if you are going to write a library in Rust, it is worth the effort to make proper enums for your errors and to return those proper enums to a user. But, when you are writing code for an application, using `anyhow` is generally a better idea to save on complexity and work. Especially because we don't need as much detail as the proper enum errors provide *most* of the time. So, just add the following line to your `Cargo.toml` dependencies (we won't actually end up using it extensively, but we'll use it for the error type in our request guard)

```toml
anyhow = "1.0"
```

Now, here's the code for the request guard.

```rust
struct AuthenticatedUser {
    user_id: i32
}

#[rocket::async_trait]
impl<'r> FromRequest<'r> for AuthenticatedUser {
    type Error = anyhow::Error;

    async fn from_request(req: &'r Request<'_>) -> request::Outcome<Self, Self::Error> {
        let cookies = req.cookies();
        let user_id_cookie = match get_user_id_cookie(cookies) {
            Some(result) => result,
            None => return request::Outcome::Forward(())
        };
        let logged_in_user_id = match user_id_cookie.value()
            .parse::<i32>() {
                Ok(result) => result,
                Err(_err) => return request::Outcome::Forward(())
            };

        return request::Outcome::Success(AuthenticatedUser { user_id: logged_in_user_id });
    }
}

fn get_user_id_cookie<'a>(cookies: &'a CookieJar) -> Option<Cookie<'a>> {
    cookies.get_private("user_id")
}
```

Now, sadly we can't just jump into applying the request guard to our requests. First, we have to update our `tasks` model and table to properly multiple users. Let's talk about it

## Updating Our Tasks

The whole point of adding in users and authentication into our app was so that multiple people could use it. Obviously, if the app just adds and removes tasks from one table, and displays every task in the table to the user, everyone would be seeing, editing, adding and removing from the same set of tasks. We don't want that. We want users to see only the tasks they've added, and we want users to only be able to add, edit or remove their own tasks. However, as our `tasks` table is now, there is no realistic way to do that. There is no way to differentiate which user added a task to the table, because we only store an `id` and the task itself in the table.

If you're sharp, you probably figured out how we're going to differentiate them. We'll add a `user_id` column to our `tasks` table, which will give us the `id` of the user that added the task. Then, we can figure out who added which tasks and thus can make the app work for multiple users. This `user_id` is called a foreign key, and relating tables to each other using foreign keys is one of the central concepts of relational databases. And, you guessed it, postgres is a relational database.

You don't need to understand how the code actually creates a foreign key, just make sure the `up` function in your `create_tasks_table` migration looks like this

```rust
async fn up(&self, manager: &SchemaManager) -> Result<(), DbErr> {
    manager
        .create_table(
            sea_query::Table::create()
                .table(Tasks::Table)
                .if_not_exists()
                .col(
                    ColumnDef::new(Tasks::Id)
                        .integer()
                        .not_null()
                        .auto_increment()
                        .primary_key(),
                )
                .col(ColumnDef::new(Tasks::Item).string().not_null())
                .col(ColumnDef::new(Tasks::UserId).integer().default(Value::Int(None)))
                .foreign_key(
                    ForeignKey::create()
                        .name("user_id")
                        .from(Tasks::Table, Tasks::UserId)
                        .to(Users::Table, Users::Id)
                        .on_delete(ForeignKeyAction::Cascade)
                        .on_update(ForeignKeyAction::Cascade)

                )
                .to_owned()
        )
        .await
}
```

Of course, you'll need to modify the `Tasks` enum in that same file to look like this to make it work

```rust
#[derive(Iden)]
pub enum Tasks {
    Table,
    Id,
    Item,
    UserId
}
```

With that setup, you can run the following command in your terminal in the project directory to modify your `tasks` table to now have a user id.

```bash
sea-orm-cli migrate fresh
```

Now, go to our `tasks` entity and modify it to look like the following

```rust
use rocket::{serde::{Deserialize, Serialize}, FromForm};
use sea_orm::entity::prelude::*;

#[derive(Clone, Debug, PartialEq, DeriveEntityModel, Deserialize, Serialize, FromForm)]
#[serde(crate = "rocket::serde")]
#[sea_orm(table_name = "tasks")]
pub struct Model {
    #[sea_orm(primary_key)]
    #[field(default = 0)]
    pub id: i32,
    pub item: String,
    #[field(default = 0)]
    pub user_id: i32
}

#[derive(Copy, Clone, Debug, EnumIter)]
pub enum Relation {
    Users
}

impl RelationTrait for Relation {
    fn def(&self) -> RelationDef {
        match self {
            Self::Users => Entity::belongs_to(super::users::Entity)
                .from(Column::UserId)
                .to(super::users::Column::Id)
                .into(),
        }
    }
}

impl Related<super::users::Entity> for Entity {
    fn to() -> RelationDef {
        Relation::Users.def()
    }
}

impl ActiveModelBehavior for ActiveModel {}

```

Finally, go to your `users` entity and make sure it looks like the following

```rust
use rocket::{serde::{Deserialize, Serialize}, FromForm};
use sea_orm::entity::prelude::*;

#[derive(Clone, Debug, PartialEq, DeriveEntityModel, Deserialize, Serialize, FromForm)]
#[serde(crate = "rocket::serde")]
#[sea_orm(table_name = "users")]
pub struct Model {
    #[sea_orm(primary_key)]
    #[field(default = 0)]
    pub id: i32,
    pub username: String,
    pub password: String
}

#[derive(Copy, Clone, Debug, EnumIter)]
pub enum Relation {
    Tasks
}

impl RelationTrait for Relation {
    fn def(&self) -> RelationDef {
        match self {
            Self::Tasks => Entity::has_many(super::tasks::Entity).into(),
        }
    }
}

impl Related<super::tasks::Entity> for Entity {
    fn to() -> RelationDef {
        Relation::Tasks.def()
    }
}

pub const USER_PASSWORD_SALT: &[u8] = b"some_random_salt";

impl ActiveModelBehavior for ActiveModel {}

```

With that sorted out, we can go back to `main.rs` and modify our CRUD operations.

## Task CRUD Operations

With each of the CRUD operations, we want to modify them to use the request guard, and to use the `user_id` returned from the request guard to add, remove, edit and view tasks from a specific user rather than just putting them in generally. Also, if the request guard fails, we want them our users to be redirected to our login page, which is where the extra routes with ranks come from. I'm just going to do all of them at once and put it into one large code block for your viewing pleasure, as this is starting to get quite tedious, and all of them will have very similar adjustments

```rust
fn redirect_to_login() -> Redirect {
    Redirect::to("/login")
}

#[post("/addtask", data="<task_form>")]
async fn add_task(conn: Connection<'_, Db>, task_form: Form<tasks::Model>, user: AuthenticatedUser) -> Flash<Redirect> {
    let db = conn.into_inner();
    let task = task_form.into_inner();

    let active_task: tasks::ActiveModel = tasks::ActiveModel {
        item: Set(task.item),
        user_id: Set(user.user_id),
        ..Default::default()
    };

    match active_task.insert(db).await {
        Ok(result) => result,
        Err(_) => {
            return Flash::error(Redirect::to("/"), "Issue creating the task");
        }
    };

    Flash::success(Redirect::to("/"), "Task created!")
}

#[post("/addtask", rank = 2)]
async fn add_task_redirect() -> Redirect {
    redirect_to_login()
}

#[put("/edittask", data="<task_form>")]
async fn edit_task(conn: Connection<'_, Db>, task_form: Form<tasks::Model>, _user: AuthenticatedUser) -> Flash<Redirect>{
    let db = conn.into_inner();
    let task = task_form.into_inner();

    let task_to_update = match Tasks::find_by_id(task.id).one(db).await {
        Ok(result) => result,
        Err(_) => {
            return Flash::error(Redirect::to("/"), "Issue editing the task");
        }
    };
    let mut task_to_update: tasks::ActiveModel = task_to_update.unwrap().into();
    task_to_update.item = Set(task.item);
    match task_to_update.update(db).await {
        Ok(result) => result,
        Err(_) => {
            return Flash::error(Redirect::to("/"), "Issue editing the task");
        }
    };

    Flash::success(Redirect::to("/"), "Task edited succesfully!")
}

#[put("/edittask", rank = 2)]
async fn edit_task_redirect() -> Redirect {
    redirect_to_login()
}

#[delete("/deletetask/<id>")]
async fn delete_task(conn: Connection<'_, Db>, id: i32, _user: AuthenticatedUser) -> Flash<Redirect> {
    let db = conn.into_inner();
    let _result = match Tasks::delete_by_id(id).exec(db).await {
        Ok(value) => value,
        Err(_) => {
            return Flash::error(Redirect::to("/"), "Issue deleting the task");
        }
    };

    Flash::success(Redirect::to("/"), "Task succesfully deleted!")
}

#[delete("/deletetask/<id>", rank = 2)]
async fn delete_task_redirect(id: i32) -> Redirect {
    redirect_to_login()
}

#[get("/?<page>&<tasks_per_page>")]
async fn index(conn: Connection<'_, Db>, flash: Option<FlashMessage<'_>>, page: Option<usize>, tasks_per_page: Option<usize>, user: AuthenticatedUser) -> Result<Template, DatabaseError> {
    let db = conn.into_inner();
    let page = page.unwrap_or(0);
    let tasks_per_page = tasks_per_page.unwrap_or(5);

    let paginator = Tasks::find()
                            .filter(tasks::Column::UserId.eq(user.user_id))
                            .order_by_asc(tasks::Column::Id)
                            .paginate(db, tasks_per_page);
    let number_of_pages = paginator.num_pages().await?;
    let tasks = paginator.fetch_page(page).await?;
    
    
    Ok(Template::render(
        "todo_list",
        json!({
            "tasks": tasks,
            "flash": flash.map(FlashMessage::into_inner),
            "number_of_pages": number_of_pages,
            "current_page": page
        })
    ))
}

#[get("/?<page>&<tasks_per_page>", rank = 2)]
async fn index_redirect(page: Option<usize>, tasks_per_page: Option<usize>) -> Redirect {
    redirect_to_login()
}

#[get("/edit/<id>")]
async fn edit_task_page(conn: Connection<'_, Db>, id: i32, _user: AuthenticatedUser) -> Result<Template, DatabaseError> {
    let db = conn.into_inner();
    let task = Tasks::find_by_id(id).one(db).await?.unwrap();

    Ok(Template::render(
        "edit_task_form", 
        json!({
            "task": task
        })
    ))
}

#[get("/edit/<id>", rank = 2)]
async fn edit_task_page_redirect(id: i32) -> Redirect {
    redirect_to_login()
}

```

Of course, make sure all the routes are added to the `rocket` function.

## Log Out Function

Now, we just need to add the functionality to log out of our app, and we can put this whole, painful authorization stuff behind us. Luckily, logging out is simple. We just have to remove the `cookie` that verifies us. But first, we need to add a button to our app that allows us to log out. So, go to `todo_list.html.tera` and add the following snippet right beneath the import of `todo_list.css`

```html
<form action="/logout" method="post">
    <button type="submit">Log Out</button>
</form>
```

After that, go into `main.rs` and add the following code, making sure to register the route in `rocket`.

```rust
fn remove_user_id_cookie(cookies: & CookieJar) {
    cookies.remove_private(Cookie::named("user_id"));
}

#[post("/logout")]
async fn logout(cookies: & CookieJar<'_>) -> Flash<Redirect> {
    remove_user_id_cookie(cookies);
    Flash::success(Redirect::to("/login"), "Logged out succesfully!")
}
```

With that, you should now be able to use multiple users in the app, and that concludes everything this beheamoth of a piece covers.

# Final Thoughts on Rocket

This was quite the journey to implement a basic `todo` app. If I had just stuck to not using the ORM, this probably would've been easier. Something like Sea ORM is useful when you have a lot of tables and data, but it's not really worth it for something as small as this. I would've had a much easier time just sticking with the database driver and writing out the queries myself. Over-engineering a solution can often make the solution harder to use, and leave the people left with your work very annoyed.

Overall, Rocket worked pretty wonderfully, even I had a few pain points when getting the ORM to work. It really is simple to setup a server using it, and its extensibility allows you to go wild with how you use that server. I think it is annoying that they don't have some basic components to use to get started with authentication, and force you to write out a lot of the authentication boiler plate other frameworks would generally help you avoid. Something like [Laravel](https://laravel.com/) does a much better job at turning authentication into less of a hassle and pain.

I'm not sure if forwarding the request guards was the right move for re-directing a user to login if they weren't authenticated. I couldn't find anyway to just redirect if the guard failed, so I ended up having to repeat the same small snippet over and over again to redirect to the login for every single route that had to be authenticated. I guess the advantage is that the logic for routing is explicit, so I could change that to be something more fancy in the future if I wanted. But, it would be nice if I could define some default redirection if a request guard fails and have written forwards overwrite that, but I guess that may be hard to implement.

Would I use it for an actual web application? Maybe, but it does seem like the code I ended up with was far more verbose than, say, a Laravel project. I feel like I could've implemented this basic CRUD example in Laravel 10 times over in the time it took me to do it in Rocket. After using Rust for this tutorial, I've fallen in love with it. Sorting out errors and writing in it has been a blast, but I'm not sure if Rocket is a good choice of framework if you are trying to produce an app in a reasonable amount of time. Maybe I'm slow, and maybe it is a great choice, I'm not entirely decided yet.

# Where to Go From Here

Now that you have a good idea of all the building blocks of web development, it may be a good idea to start developing your own web apps. As you run into problems, you'll probably Google them and you can start to fill out the gaps in knowledge I left here. If you wanted to learn even more about web development even quicker, use another web framework like [Laravel](https://laravel.com/) or [Spring](https://spring.io/). Seeing slightly different interpretations of these concepts, and other useful features can teach you even more about the web development space.

However, maybe you are just trying to bolster your development skills. In that case, there are plenty of books that can help you strengthen your code writing skills in general. I recommend [Clean Code](https://github.com/jnguyen095/clean-code/blob/master/Clean.Code.A.Handbook.of.Agile.Software.Craftsmanship.pdf), [Task-Centered User Interface Design](http://hcibib.org/tcuid/tcuid.pdf), [Code Complete, Second Edition](https://khmerbamboo.files.wordpress.com/2014/09/code-complete-2nd-edition-v413hav.pdf) and [The Mythical Man Month](https://web.eecs.umich.edu/~weimerw/2018-481/readings/mythical-man-month.pdf). Finally, I would be remissed and probably lynched if I didn't at least mention [Design Patterns: Elements of Reusable Object-Oriented Software](https://github.com/amilajack/reading/blob/master/Design/GOF%20Design%20Patterns.pdf). I'm not sure if design patterns are all they're cracked up to be, but being aware of them will make you better at communicating to your fellow developers. You shouldn't follow any of these book's advice dogmatically, and you don't have to read all of these books cover-to-cover, but they can give you some useful ideas on how you should think of code and what "good code" looks like. They are especially useful for beginning developers.

If you want books that provide more concrete information that could help you improve, I would suggest [Effective Rust](https://www.lurklurk.org/effective-rust/cover.html) and [Effective Java](https://kea.nu/files/textbooks/new/Effective%20Java%20%282017%2C%20Addison-Wesley%29.pdf). Of course, neither are the final say on what good code looks like, but they give you a lot of ideas for improving your code in either language.

There are many other materials out there, so if you are not interested in any of the resources I've offered here, do some Googling and find your own technical resources that you think look promising. As long as you commit to yourself to becoming a better developer, and take sincere and honest steps towards bettering yourself, you will become a good developer in no time.

Well, that's all. I hope I taught you something in this mess of 21,000 words. Believe it or not, there are still A LOT of concepts in web development that I didn't even touch on here. There's a large sea of knowledge when it comes to building web apps, but hopefully I've given you enough information so that you can at least start building your own web apps. Good luck!