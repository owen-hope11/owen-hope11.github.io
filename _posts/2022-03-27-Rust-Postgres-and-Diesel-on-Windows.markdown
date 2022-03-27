# Rust Postgres and Diesel on Windows

## Background
 * Add point to skip to here.

After deciding to start a side project my close friend and I have been talking about for a while and finally getting some downtime to do it, I figured I'd get it up off the ground using the Rust web eco system. **What a struggle that was!** I was introduced to Rust about 3 years ago at my first job right out of college. I fell in love with it instantly and began to absorb as much knowledge as I could about it. At the time I was working on two projects in two different languages - one being C++ and the other being Rust, both dealing in low level networking system applications (Routing layer and below). This gave me so much insight on how there were pitfalls in C++ when it came to memory issue (seg faults, memory leaks :/ ya know... the really fun stuff to debug) and how Rust can ease that pain. Rust was also so new and fresh it was exciting to learn!

In the beginning of October I joined a new company as a backend software engineer. They were using Python in their backend with the hope of starting to build some systems out in Rust. We aren't quite there yet, but there is a lot of pro-Rust support so I still have confidence we'll get there! But since I've left my previous company I've been itching to start writing in Rust again! Which brings me to how I stumbled upon this frustrating journey of how to set up your development environment for Rust web development on Windows.

### You might start caring about stuff at this point

Developing on Windows has just never been my thing - everything just seems 100x easier to set up on a \*nix machine. I mainly have used Ubuntu or MacOS (work laptop). But this time I had a brand new PC rig running Windows that just seemed like a shame not to use. I'm sure some of you are thinking "just dual boot it to linux" and to that I say yes that would make sense, but I just wanted to get this up and running as a weekend project. Thinking how hard could it really be to get setup on Windows really? Well I found out f***ing hard.

## The challenges and errors I faced (You care about this)

* Getting a local Postgres database running
* Missing .dll files (api-ms-win-crt-*.dll)
* Logging into the database with the incorrect username due to `ident` being used.
    * `ident` - Obtain the operating system username of the client by contacting the ident server on the client and check if it matches the requested database username.
* Username not in Database.
* Password incorrect.
* Do not have permissions to create Database.

## Installing Postgres

Follow this guide [here](https://www.postgresqltutorial.com/install-postgresql/). It will walk you through the very basic steps of just getting Postgres on your Windows machine. There is a lot more work to do after that.

**Note: anything inside of `<>` should be replaced and the arrows `<>` should be removed as well.**

After installing Postgres from the link above, you are going to want to add some environment variables to your system.

* In the windows search bar type "View advanced system settings". After you select it, what comes up should look like this:

![image](/photos/advanced_system_settings.png)

* In the bottom right corner, select "Environment Variables..." and another window should pop up:

![image](/photos/env_variables.png)

* Now you will want to add two environment variables
    * `PQ_BIN_DIR` - The path for this should be `C:\Program Files\PostgreSQL\<version # you installed>\bin` 
    * `PQ_LIB_DIR` - The path for this should be `C:\Program Files\PostgreSQL\<version # you installed>\lib`

**For the rest of this guide I will assume you have git and git-bash installed. If you don't, I highly suggest you install it [here](https://git-scm.com/download/win).**

### Getting Postgres to run on git bash

* You will need to add the postgreSQL bin to your path so you can use the `psql` command. Since you'll want this command to always be available, we are going to put it in the `.bashrc` file.
    * Move to your root directory - a simple `cd` should bring you there.
    * Unless you have already added one, there shouldn't be a `.bashrc` file. Create one with the editor of your choice. Paste in: `PATH=$PATH:/c/"Program Files"/PostgreSQL/<version # you installed>/bin`
    * Close out your terminal session and re-open it for this to take effect.
    * You should now be able to use the `psql` command.

### Setting up your Postgres user
If you followed the tutorial in the [link](https://www.postgresqltutorial.com/install-postgresql/) above on how to install Postgres, this creates a user with the username `postgres` and a password that you entered. When Postgres is installed, it usually sets the authentication method to use `ident` which takes your OS username and uses it as the Postgres username to log in with by default. Instead of playing around with the `pg_hba.conf` file (more info [here](https://www.postgresql.org/docs/9.1/auth-pg-hba-conf.html) if you want it). We are going to just add ourselves as a user.

* To get what Postgres thinks is your OS username, just run `psql` and it should display it for you.
* Login as the `postgres` user, since this is the only one currently available and it has a bunch of Role attributes.
    * `psql -U postgres`
    * Enter your password that you choose.
* Create your database.
    * `CREATE DATABASE <user_name shown from above>;`
* Create your user using SQL commands.
    * `CREATE USER <user_name shown from above> WITH ENCRYPTED PASSWORD '<enter your password>';`
* Add Attributes to your role that you just created to match the `postgres` user's roles
    * `ALTER ROLE <user_name> WITH SUPERUSER CREATEROLE CREATEDB REPLICATION BYPASSRLS;`
* Exit out of the `postgres` user and try to login with your user by running
    * `psql` which should prompt you for your password.

## Setting up Diesel

With all of that taken care of, now we’ll start getting into the Rust side of things. 

**I'm going to assume you have Rust installed already. If not here's the [link](https://www.rust-lang.org/tools/install)**

### Installing Diesel's cli
Setting this up was the biggest pain-point throughout this whole process, so hopefully this will make it easier for you.

For reasons that are still unknown to me, the Diesel cli installation kept failing due to being unable to find multiple `.dll` files. The solution to this was to copy them into my `~/.cargo/bin` folder. If someone does have an idea of why this might be I'd love to know - a working theory is that it has something to do with `rustc` linkage.

Files needing to be copied:
* The database `.dll` file. In this case we are using Postgres, so it was `libpq.dll`
* `api-ms-win-crt-heap-l1-1-0.dll`
* `api-ms-win-crt-locale-l1-1-0.dll`
* `api-ms-win-crt-math-l1-1-0.dll`
* `api-ms-win-crt-runtime-l1-1-0.dll`
* `api-ms-win-crt-stdio-l1-1-0.dll`
* `api-ms-win-crt-string-l1-1-0.dll`

The `libpq.dll` file can be found at `C:\Program Files\PostgreSQL\14\bin\libpq.dll`

As for all the `api-ms-win-crt` files, this took some time to find. But they are all in the  `C:\Windows\System32\downlevel\` directory. If you don't see them, it's possible they are hidden. To show them:
* Open file explorer and navigate to that directory.
* Right-click on an empty space and select `Properties` all the way at the bottom.
* In the window that comes up, there should be a checkbox next to the word Hidden.
    * Select that checkbox
    * Click `Apply`

To install the Diesel cli to use Postgres, run:
* `cargo install diesel_cli --no-default-features --features postgres`
* Running `diesel` in your terminal should show you all the options.

## Final Product
To test that this is working, create a quick rust test program.
* `cargo new test_diesel --bin`

Move into the project directory
* `cd test_diesel`

Open up the `Cargo.toml` folder in the editor of your choosing and add:
```toml
[dependencies]
diesel = { version = "1.4.4", features = ["postgres"] }
dotenv = "0.15.0"
```

Diesel need to know where to look for our Postgres database, so we want to set an environment variable of `DATABASE_URL`. We can place this in a `.env` file so our system variables do not get over-crowded.
```
echo DATABASE_URL=postgres://username:password@localhost/<name of database you want to create> > .env
```
*Note:* If you have special characters in your password, this won't work in the terminal. You should run: `touch .env`to create the `.env` file, then open it and enter the environment variable there.

Now you can run:
* `diesel setup` to create the database.

Then to create the table:
* `diesel migration generate test_tabel`

This should create two files in `migrations/<date>_<number>_<table_name>` called `down.sql` and `up.sql`.

Once you see these, you know Diesel and Postgres are running correctly with each other!

Now with both Postgres setup and the Diesel cli you should be able to get a Rust web backend up and running.
