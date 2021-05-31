# Creating Shell Scripts

Sometimes, you'll need to write a shell script. This is a file that is executed by a command in your terminal. So, for example, we've run a command like `python3 file_name.py`, which just means that we're using the `python3` command to execute the code in a file called `file_name.py`.

It's the same thing with `node fileName.js`. We are using the `node` command to execute the code in a file called `fileName.js`. So far, the code that we've been executing has consisted of combinations of control flow and `console.log` or `print` methods in order to practice fundamental programming concepts.

But, what if you could create a file, or **script**, that takes in arguments that follow the file path? In other words, what if we ran a command like `python3 say_hello.py Maria Lopez` so that the `say_hello.py` file actually grabbed `Maria` and `Lopez` and used those arguments in its logic?

## Command Line Arguments

When we run commands, we are usually passing arguments to that command. For example, when we run `git push origin master`, we are passing `push` and `origin` and `master` to the `git` command. These arguments can be other commands, file names, general values, or other possibilities.

When we run `npm install express`, we are passing and additional command called `install` to the `npm` command, along with the name of the package that we want to install (e.g., `express`).

## Argument Vector

In most programming languages, there is a special variable called `argv`, which stands for **argument vector**. Here's an explanation from a [Flatiron School](https://flatironschool.com/blog/a-short-explanation-of-argv) article:

> It refers to the "argument vector," which is basically a variable that contains the arguments passed to a program through the command line.

Basically, any string of arguments passed to a command can be captured into an array (or list) of strings. Then, we can use that array/list within our script's logic.

## Setup

1. `cd ~/code`
2. `mkdir shell-scripts && cd shell-scripts`
3. `touch python_script.py`
4. `code .`

## Accessing Command Line arguments

In order to use the `argv` variable, we'll need to use Python's `sys` module, so inside `python_script.py`, let's write:

```python
import sys

print(sys.argv)
```

Now, in your terminal, run:

```python
python3 python_script.py Hello World
```

Thanks to the `sys.argv` variable, we now have access to any arguments we might need to pass into our `python_script.py` script.

Because you can access the values of `sys.argv` like you would a list, see if you can get your script to interpolate the arguments into a string so that the script prints out `Hello World`.

<details>
  <summary>Hello World</summary>

```python
print(f"{sys.argv[1]} {sys.argv[2]}")
```

</details>

As you can imagine, you can use these arguments for a variety of purposes: to determine a condition in an if/else statement, to pass into a function, to use as a condition of a loop, and so on.

## Why Shell Scripts?

When would we actually use shell scripts? There are [many uses](https://bash.cyberciti.biz/guide/Why_shell_scripting), but one example might be consuming data from a file and inserting it into your database.

Imagine that you've taken on a new client and you're creating an app for them. They've built up years of data, which they've exported to CSV files.

You could create a feature in their app to allow them to upload their files and be automatically consumed so that the data is written to your database. But, if this action won't need to be performed after the first time, then creating an entire UI would be overkill.

Instead, you can write a script that consumes their files and maps their data to your own tables.

Let's simulate this.

## Consuming a CSV

Create a file called `monty_python.csv`. Copy and paste the following data into it:

```
first_name,last_name,age,education,is_alive
Graham,Chapman,48,Cambridge,no
John,Cleese,80,Cambridge,yes
Terry,Gilliam,79,Occidental,yes
Eric,Idle,76,Cambridge,yes
Terry,Jones,77,Oxford,no
Michael,Palin,76,Oxford,yes
```

CSV stands for "comma-separated values," which is why those commas in the data are so important. They will tell our program where each value begins and ends. And, each line break will tell our program when the row is complete.

Notice that the first row doesn't actually contain any data. These are our headers that we'll use to map the subsequent data.

We'll need Python's `csv` module to work with this file type, let's include the `import` toward the top:

```python
import sys
import csv
```

Our only argument will be the `monty_python.csv` file, so let's set that to a variable called `imported_file`:

```python
import sys
import csv

imported_file = sys.argv[1]
```

In the next block of code, we'll use functionality from that `csv` module, though we won't go into detail about the individual methods:

```python
import sys
import csv

imported_file = sys.argv[1]

with open(imported_file, newline='') as csvfile:
    reader = csv.DictReader(csvfile)
    for row in reader:

        print(f"{row['first_name']} {row['last_name']} {row['age']} {row['education']} {row['is_alive']}")
```

If you would like to read more about the details of the `with` statement and the `csv.DictReader`, checkout the [Python documentation](https://docs.python.org/3/library/csv.html).

The gist of what's happening is that we're opening the `imported_file`, which is then being fed into the `csv` module's `DictReader` method, so that we can iterate through the data.

Finally, we're iterating through the dictionary that was created from the data and printing out each of the values of each row.

Now, let's run `python3 python_script.py monty_python.csv` and see what happens.

See if you can create another `.csv` file with different data. Then, try passing that file into your script.

## Writing the Data into a PostgreSQL Database

Printing out the data from a file is fantastic! But, ultimately, it probably makes sense to take the csv data and write it to a database so that it can be used by other parts of an app.

### Setting up the Database

1. `psql`
2. `CREATE DATABASE monty_python;`
3. `\c monty_python`
4. 
```sql
CREATE TABLE members (
    id SERIAL,
    first_name varchar(24),
    last_name varchar(24),
    age integer,
    education varchar(24),
    is_alive integer DEFAULT 1
);
```

### Connecting to the Database from the Script

1. At the top of the file, import `psycopg2`:

```python
import sys
import csv
import psycopg2
...
```
2. Underneath the imports, establish connection to the db, and open a cursor:

```python
conn = psycopg2.connect("dbname=monty_python user=your_postgres_username password=your_postgres_password")
cur = conn.cursor()
```

If you're not sure what your postgres username is you can go into `psql` and run `\du`, which will list the users.

As for the password, it may be something as simple as `postgres` or `root`. If it's not and you don't remember what it is, you can reset it using [this method](https://stackoverflow.com/questions/15008204/how-to-check-postgres-user-and-password) or [this method](https://serverfault.com/questions/110154/whats-the-default-superuser-username-password-for-postgres-after-a-new-install), though I don't know that it's important enough to mess with that for this exercise.

3. Inside the loop, convert data, if necessary. Let's convert 'yes' and 'no' to 1s and 0s:

```python
is_alive = 1 if row['is_alive'] == 'yes' else 0
```

4. Inside the loop, build and execute the query, and then save changes to the db:

```python
cur.execute(
    f"INSERT INTO members (first_name, last_name, age, education, is_alive)
    VALUES('{row['first_name']}', '{row['last_name']}', '{row['age']}', '{row['education']}', {is_alive});"
)
conn.commit()
```

5. At the bottom of the file, close the cursor and close the connection to the db:

```python
cur.close()
conn.close()
```

This is what the entire file should look like:

```python
import sys
import csv
import psycopg2

# connect to db using credentials
conn = psycopg2.connect("dbname=monty_python user=your_postgres_username password=your_postgres_password")
# create a cursor to traverse the rows of a result set
cur = conn.cursor()

imported_file = sys.argv[1]

with open(imported_file, newline='') as csvfile:
    reader = csv.DictReader(csvfile)
    for row in reader:
        # the is_alive column uses 1 or 0 to indicate yes/no
        is_alive = 1 if row['is_alive'] == 'yes' else 0
        # build and execute query
        cur.execute(
            f"INSERT INTO members (first_name, last_name, age, education, is_alive) VALUES('{row['first_name']}', '{row['last_name']}', '{row['age']}', '{row['education']}', {is_alive});"
        )
        # save changes to db
        conn.commit()

        print(f"{row['first_name']} {row['last_name']} {row['age']} {row['education']} {is_alive}")


# close the cursor for the sake of other queries
cur.close()
# close the connection to the db
conn.close()
```

Let's run `python3 python_script.py monty_python.csv` again.

Now go into your database and query your `members` table:

1. `psql`
2. `\c monty_python`
3. `SELECT * FROM members;`

You've now taken data from a file and written it to a database!

## Other Programming languages

As mentioned before, writing shell scripts can be done in just about every programming language. For example, if you'd like to create one in JavaScript, you can use the `process.argv` variable and you don't even need to import any libraries or packages.