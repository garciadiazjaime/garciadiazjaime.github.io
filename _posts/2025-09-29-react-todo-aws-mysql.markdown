---
layout: post
title: "Nextjs Todo Application Using Mysql and Terraform"
date: 2025-09-29 10:00:00 -0500
categories: nextjs aws mysql terraform
---

![Nextjs Todo Application Using Mysql and Terraform](/assets/react-todo-aws-mysql/banner.png)

Hi, here’s another Todo application in React. Nothing exceptional compared to other examples, but I’ll focus more on hosting the database in AWS and using Terraform for deployment.

Take a look at the [Demo](https://demo.garciadiazjaime.com/react-todo-aws-mysql) and [Codebase](https://github.com/garciadiazjaime/demo-reactjs/blob/main/app/react-todo-aws-mysql/page.tsx).

## Front-End Architecture

![Todo List Architecture](/assets/react-todo-aws-mysql/architecture.jpeg)

The application can perform six main functions:

- Add Todos
- Edit a Single Todo
- Delete a Single Todo
- Get all Todos
- Delete all Todos

As a good practice for client applications, all user interactions are first stored in the browser — in this case, in a React state variable — and then synced with the database in the background.

The downside of this approach is the risk of a race condition. For example, if the user closes the browser before the todo is saved in the database, the information will be lost. The upside is that the user doesn’t need to wait for backend confirmation, which makes the app feel more responsive. And since users typically don’t close the app immediately, this trade-off works fine for most cases.

For this demo, the goal is not to cover 100% of scenarios but to show the overall approach.

## Snippets

In the codebase you will find things like this:

```javascript
const newTodos = textareaValue
  .split("\n")
  .filter((line) => line.trim() !== "")
  .map((line, index) => ({
    id: Date.now() + index,
    text: line.trim().toLocaleLowerCase(),
    done: false,
  }));

setTodos(newTodos);

await addTodosToDB(newTodos);
```

Notice how the React state variable `todos` is updated. This triggers a change in the UI, allowing the user to see the new todos in the list. Meanwhile, in the background, `await addTodosToDB(newTodos)` is executed to save the `todos` in the database.

Let’s take a look at the function [addTodosToDB](https://github.com/garciadiazjaime/demo-reactjs/blob/main/app/react-todo-aws-mysql/todoActions.ts#L31):

```javascript
"use server";

...

export async function addTodosToDB(
  todos: { id: number, text: string, done: boolean }[]
) {
  const values = todos.map((todo) => [todo.id, todo.text, todo.done, LIST]);
  await db.query("INSERT INTO todos (id, text, done, list) VALUES ?", [values]);
}
```

Here, you’ll notice `"use server";` at the top of the file. This tells Next.js that the file should be executed on the server as a [server action](https://www.garciadiazjaime.com/posts/react-19-server-function).

Since the code runs on the server, it’s fine to interact directly with the database—for example, with a raw MySQL insert query.

> In a production application, you’ll usually want to use an ORM when working with a database. ORM packages typically provide features such as migrations, validation, schemas, and more.

I won’t go into the rest of the code since it’s fairly self-explanatory. Also, most of it was generated with the help of an AI agent. Here are some of the prompts I used:

```
Create a react component for a todo list application that allows users to add, edit, delete, and mark todos as done. The todos should be fetched from a database, and changes should be synced back to the database. Include a textarea for bulk adding todos and a reset button to clear the list.
```

```
Add functionality to a todo list component where pressing 'enter' adds a new todo, 'shift + enter' allows multiline input, and 'escape' clears the textarea. Include buttons for toggling the 'done' state, editing todos inline, and deleting todos. Ensure the UI updates dynamically and is synced with a backend database. Style the component with inline styles for a clean and modern look.
```

## Terraform (Database)

This is the part I’m most interested in. As a web developer, it’s pretty easy to build a web application—especially now with AI agents—but deploying it is not always as straightforward. There are some cool [services that let you host applications for free](https://www.garciadiazjaime.com/posts/web-application-free-hosting), and I definitely recommend them. However, in most work environments you’ll run into AWS, especially for databases.

> The following steps will create a MySQL database in AWS and add a network so the database can be accessed from the application.

Here’s what you’ll need:

1. An AWS account (credit card required)
2. Access Keys, which consist of a Key ID and a Secret Access Key

![AWS Key ID and Secret Access Key](/assets/react-todo-aws-mysql/aws-key-access.png)

3. [Terraform](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli) installed

These first three steps might take a bit of time, but once you have them, the next step is:

4. **AWS Profile**

Not required, but it’s a good way to manage your credentials in your local environment.

```sh
$ aws configure --profile todo-profile

AWS Access Key ID [****************]: ****************
AWS Secret Access Key [****************]: ****************
Default region name [None]:
Default output format [None]:
```

5. Environment Variables

You’ll need to `export` the following variables in your terminal:

```sh
export TF_VAR_mysql_database="*****"
export TF_VAR_mysql_user=*****
export TF_VAR_mysql_pwd="*****"
```

**Note: The values are whatever string you want.**

In the Terraform `rds` file, these variables are used like this:

```sh
db_name                 = var.mysql_database  # Pull from environment variable
username                = var.mysql_user
password                = var.mysql_pwd
```

Terraform automatically picks up any environment variable that’s prefixed with `TF_VAR_` and will map these to the corresponding variables defined in your `variables.tf` file.

5. Init

```sh
$ terraform init

Initializing the backend...
Initializing provider plugins...
- Reusing previous version of hashicorp/aws from the dependency lock file
- Reusing previous version of hashicorp/null from the dependency lock file
- Using previously-installed hashicorp/aws v6.13.0
- Using previously-installed hashicorp/null v3.2.4

Terraform has been successfully initialized!

...
```

6. Plan

```sh
$ terraform plan

data.aws_availability_zones.available: Reading...
data.aws_availability_zones.available: Read complete after 0s [id=us-east-1]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_db_instance.todo_db will be created
  + resource "aws_db_instance" "todo_db" {
      + address                               = (known after apply)
      + allocated_storage                     = 20
...

Plan: 12 to add, 0 to change, 0 to destroy.
```

7. Apply

```sh
terraform apply
data.aws_availability_zones.available: Reading...

...

Plan: 12 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_vpc.todo_vpc: Creating...
aws_vpc.todo_vpc: Still creating... [10s elapsed]
aws_vpc.todo_vpc: Creation complete after 11s [id=vpc-095daafd54946d1c0]

...

null_resource.init_db (local-exec): host: *****

...

Apply complete! Resources: 12 added, 0 changed, 0 destroyed.
```

---

Notice the line above that shows the `host` URL:

```sh
null_resource.init_db (local-exec): host: *****
```

This is important because the application will need it. You can also use it to connect directly to your database.

Just type your password when prompted, and you should be inside AWS.

```sh
mysql -h [host] -P 3306 -u [user] -p
Enter password:
Welcome to the MySQL monitor.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| todo_database      |
+--------------------+

mysql> use todo_database;
Database changed

mysql> show tables;
+-------------------------+
| Tables_in_todo_database |
+-------------------------+
| todos                   |
+-------------------------+
```

Perfect! You can also view your MySQL database in the AWS UI.

![AWS RDS Database MySQL](/assets/react-todo-aws-mysql/aws-mysql-database.png)

## Webapp Database Credentials

This is how the application receives its credentials:

```javascript
const db = mysql.createPool({
  host: process.env.TF_VAR_mysql_host,
  user: process.env.TF_VAR_mysql_user,
  password: process.env.TF_VAR_mysql_pwd,
  database: process.env.TF_VAR_mysql_database,
});
```

I added one extra variable, `TF_VAR_mysql_host`, whose value you get after applying the Terraform scripts.

## Costs

Using AWS is usually not free. Here’s the cost of running this small database for almost a month:

![AWS Relational Database Service and Virtual Private Cloud Cost](/assets/react-todo-aws-mysql/aws-mysql-costs.png)

The Virtual Private Cloud (VPC) provides access via HTTP to the database, which is necessary for connecting the web application to it.

One nice thing about using `Terraform` is that you can remove everything from AWS just by running:

```sh
$ terraform destroy

data.aws_availability_zones.available: Reading...
...

Plan: 0 to add, 0 to change, 12 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

null_resource.init_db: Destroying... [id=5959882105264964999]
...

Destroy complete! Resources: 12 destroyed.
```

By destroying everything, you stop AWS from charging you—especially if it’s just for testing. When working for a company, it’s also a good practice to remove unused resources to reduce costs.

## Final Thoughts

As a web developer, at some point you’ll likely need a database. In a work environment, this usually means setting one up in AWS. Feel free to experiment with the demo: the Next.js application has both a client side and a server side. The server side (server actions) is integrated directly with the MySQL database hosted in AWS.
