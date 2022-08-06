## Creating Polymorphic Relationships using Node.js and Sequelize

In my last [article](https://blogs.aayushkurup.dev/polymorphic-relationships-a-database-relationship-your-college-probably-wont-teach-you), I explained what are polymorphic relationships. This blogs is a continuation of the same where I will demonstrate how to create a polymorphic relationship in node.js using the popular ORM [sequelize](https://sequelize.org/).

Sequelize is one of the most popular Object Relational Mapper or ORM for SQL databases, written in node.js. It has more than 26k+ stars on github, and is used by some of the best companies in the world including Walmart Labs, Uphold and Snaplytics. 

For this blog, we will first create a database models using Sequelize, then we will create an association between these models, and then finally, we will load the data using these associations. So without any further delay..... Let's get started.

# Project Setup

First, we need to initialize our project. For that, we simply run the init command in our project directory as follows:-


```
npm init
``` 

or if you prefer yarn:-

```
yarn init
```

Now, we need to install the dependencies. For this project, we will be using Sequelize. Also we will be using a postgres database, so we need to install the it's driver. If you are using some other database like MySQL or MariaDB, make sure you install the correct driver for it. We will also use the dotenv package to store and retrieve environment variables. The command to install the dependencies:-
 
```
npm install --save sequelize pg dotenv
```

# Connecting to the database

First and foremost, we will create a *.env* file to store all the database credentials.


```
/.env

DB_NAME=<Your DB name>
DB_USER=<Your DB user>
DB_PASSWORD=<Your DB Password>
DB_HOST=<Your DB hostname>
DB_PORT=<Your DB port>
``` 

Create the file and replace the values with your database credentials. Now, to load these values into your environment variables, we will use the dotenv package as follows.


```
./index.js

require('dotenv').config()
``` 

Finally, lets connect to our postgres database using sequelize:-

```
./db.js

const { Sequelize } = require('sequelize');

const conn = new Sequelize();

module.exports = conn
```
# Creating Sequelize models

Lets create Sequelize models for our tables. We will create 3 models - Image, Video and Comment. Both images and videos can have comments. So comments will be a polymorphic table. Lets define our three models:-

```
/models/Image.js

const Sequelize = require('sequelize');

const conn = require('../db');

const Image = conn.define('images', {
    id: {
        type: Sequelize.INTEGER,
        autoIncrement: true,
        allowNull: false,
        primaryKey: true
    },
    name: {
        type: Sequelize.STRING,
        allowNull: true,
    },
    url: {
        type: Sequelize.STRING,
        allowNull: false,
    }
});

module.exports = Image;
```

```
/models/Videos.js

const Sequelize = require('sequelize');

const conn = require('../db');

const Video = conn.define('videos', {
    id: {
        type: Sequelize.INTEGER,
        autoIncrement: true,
        allowNull: false,
        primaryKey: true
    },
    name: {
        type: Sequelize.STRING,
        allowNull: true,
    },
    url: {
        type: Sequelize.STRING,
        allowNull: false,
    }
});

module.exports = Video;
```

```
/models/Comment.js

const Sequelize = require('sequelize');

const conn = require('../db');

const uppercaseFirst = str => `${str[0].toUpperCase()}${str.substr(1)}`;
class Comment extends Sequelize.Model {
    getCommentable(options) {
      if (!this.commentableType) return Promise.resolve(null);
      const mixinMethodName = `get${uppercaseFirst(this.commentableType)}`;
      return this[mixinMethodName](options);
    }
}
Comment.init({
    id: {
        type: Sequelize.INTEGER,
        autoIncrement: true,
        allowNull: false,
        primaryKey: true
    },
    commentableType: {
        type: Sequelize.STRING,
        allowNull: false,
    },
    commentableId: {
        type: Sequelize.INTEGER,
        allowNull: false,
    },
    headline: {
        type: Sequelize.STRING,
        allowNull: false,
    },
    body: {
        type: Sequelize.STRING,
        allowNull: true,
    },
    likes: {
        type: Sequelize.INTEGER,
        allowNull: true,
    },
}, { sequelize: conn, modelName: 'comments' });

Comment.addHook("afterFind", findResult => {
    if (!Array.isArray(findResult)) findResult = [findResult];
    for (const instance of findResult) {
      if (instance.commentableType === "image" && instance.image !== undefined) {
        instance.commentable = instance.image;
      } else if (instance.commentableType === "video" && instance.video !== undefined) {
        instance.commentable = instance.video;
      }

      delete instance.image;
      delete instance.dataValues.image;
      delete instance.video;
      delete instance.dataValues.video;
    }
});

module.exports = Comment;
```

Nothing fancy here, I have simply defined three tables here. 

But before we create the associations, let's have a closer look at our polymorphic table, i.e. the ```Comments``` model. I have defined a ```getCommentable``` method in the Comment class. The purpose of this method is to fetch the correct mixin for the commentable type under the hood. [Mixins](https://sequelize.org/docs/v6/core-concepts/assocs/#special-methodsmixins-added-to-instances) are basically methods that models acquire when they are in an association. I have also added a hook in the ```Comment``` model, which automatically adds comment field in every instance.

# Creating the associations

Well all the heavy lifting was already done while defining the models. The hook and the getCommentable method in ```Comment``` will handle most of the heavy lifting. All we need to do now is create a one to many association between ```Video``` and ```Comment``` and also between ```Image``` and ```Comment```. Add the following lines:-

```
/index.js

Image.hasMany(Comment, {
    foreignKey: 'commentableId',
    scope: {
      commentableType: 'images'
    }
});
Comment.belongsTo(Image, { foreignKey: 'commentableId' });

Video.hasMany(Comment, {
    foreignKey: 'commentableId',
    scope: {
      commentableType: 'videos'
    }
});
Comment.belongsTo(Video, { foreignKey: 'commentableId' });
```

That's it. Now we can query data from these models as follows:-

```
const videos = await Video.findAll({ include: Comment });
```

The result of this query will be:-

```
{
    "id": 1,
    "name": "Video 1",
    "url": "Video Url 1",
    "createdAt": "2022-08-06T05:16:01.700Z",
    "updatedAt": "2022-08-06T05:16:01.700Z",
    "comments": [
        {
            "id": 2,
            "commentableType": "videos",
            "commentableId": 1,
            "headline": "Really cool video",
            "body": null,
            "likes": 0,
            "createdAt": "2022-08-06T05:16:01.711Z",
            "updatedAt": "2022-08-06T05:16:01.711Z"
        },
        {
            "id": 1,
            "commentableType": "videos",
            "commentableId": 1,
            "headline": "Nice video",
            "body": null,
            "likes": 0,
            "createdAt": "2022-08-06T05:16:01.711Z",
            "updatedAt": "2022-08-06T05:16:01.711Z"
        }
    ]
}
```

# Fin

Thanks for reading the blog. You can get all the code I used in this blog right [here](https://github.com/AayushK47/polymorphic-example). If you want to learn more about sequelize, you can check out their [website](https://sequelize.org/docs/v6/advanced-association-concepts/polymorphic-associations/).