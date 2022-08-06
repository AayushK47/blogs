## Querying Firebase Realtime Database and Cloud Firestore from your terminal

I believe that the way we all learn writing queries for a database is quite similar. After learning the basics, we pull up our terminal, start the database server and practice writing different queries. Apart from learning, a database shell also acts as a very good testing tool. Most of the databases provides us with an interface so that we can learn, except **Firebase databases**.

![Firebase](https://dev-to-uploads.s3.amazonaws.com/i/gchk8fhts5e717g3nzhs.png)

When I first used realtime database, the fact that I couldn't double check the output of my query really bugged me. So I decided to create a solution for this - **[Fireshell](npmjs.com/package/fireshell)**.

# Getting started with fireshell

[Fireshell](npmjs.com/package/fireshell) is a CLI tool which can be used to execute realtime database and cloud firestore queries in your terminal.

### Installing the package

To install fireshell, just run the following command:-

```
npm install -g fireshell
```

Make sure you have node.js and npm installed on your system before running this command.

### Connecting the shell with the database

To start the shell, simply run `fireshell` in your terminal. You will be prompted a few questions.

The shell will first ask you to select a database:-

```
? Choose one of the following (Use arrow keys)
> Realtime Database
  Cloud Firestore
```
Then you have to provide the **absolute path** to your firebase config file. It has to be a JSON file that you recieve from firebase to connect your application with your firebase project.

```
? Enter the absolute path to firebase config file
> /root/path/to/your/config.json
```

Finally, you have to provide the URL of firebase realtime database. If you are connecting to realtime database, you need to provide this url. But if you are trying to connect to firestore, you can ignore it.

```
? Enter the URL of firebase realtime database. (Ignore if you chose cloud firestore)
> https://<YOUR FIREBASE PROJECT NAME>.firebaseio.com/
```

Once these inputs are provided, the shell will be connected to your database.

### Writing Queries

Your queries must start with the keyword `db`. This `db` is a variable that stores reference to the database object. You can chain the rest of your query as you normally do.

For realtime database, make sure that you end any read query or any query that returns some data with the once method and pass value as its argument.

Some basic examples on writing queries are provided [here](https://github.com/AayushK47/fireshell/blob/master/README.md#writing-queries).

# Final Words

Thank you for checking out this blog article. Do try out fireshell and share your experience. If you face any issue or if you want to make some contribution to this project, head on to the [github repo](https://github.com/AayushK47/fireshell) and create an issue. 

Happy learning
Ciao!