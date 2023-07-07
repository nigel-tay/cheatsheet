# Cheatsheet Table of Contents:
<a href="https://github.com/nigel-tay/cheatsheet/blob/main/README.md#deploying-a-mongo-springboot-app-to-railway">Deploying a mongo springboot app to railway</a>
<a href="https://github.com/nigel-tay/cheatsheet/blob/main/README.md#paf-commands">PAF Commands</a>

# Deploying a mongo springboot app to railway
### Deploy your Springboot application to Railway
1. If you havent logged into railway locally, do it through your terminal

   `railway login`

2. Initialise a new railway project locally

   `railway init`


   You will be prompted to give a name, you can give your project any name

   `example project name`

3. Deploy it to railway

   `railway up`

---
### Provision and link a mongodb instance
1. Head over to <a href="https://railway.app/dashboard">railway</a> on your browser, and head to the dashboard
2. Click into your freshly deployed application and click on: "+ New" button > Database > add MongoDB

   This will spin up a mongodb instance
   
4. After its done, click on the MongoDB service and go into the "Connect" tab, copy the command under "Raw mongo command".

   The command should look something like this:
   
   `mongosh "mongodb://mongo:zafDXoaiuTw80q3bMvqM@containers-us-west-207.railway.app:7018"`

   We really only need the part within the inverted commas:
   
   `mongodb://mongo:zafDXoaiuTw80q3bMvqM@containers-us-west-207.railway.app:7018`
   
5. Now exit the MongoDB service and click on your application service, create a new environment variable under the "Variables" tab.

   The variable name will be `SPRING_DATA_MONGODB_URI`, and its value will be the previously copied command with some extra stuff appended to the back.

* Append the following to the command you copied previously (authSource is **CASE SENSITIVE** please take note):

   /`databaseName`?authSource=admin

   It should look something like this after you are done:

   VARIABLE_NAME: `SPRING_DATA_MONGODB_URI`
   
   VALUE: `mongodb://mongo:zafDXoaiuTw80q3bMvqM@containers-us-west-207.railway.app:7018/NAMEOFMYDATABASE?authSource=admin`

### Import your data to your MongoDB instance
To make things easier, `cd` into the directory where your data is located. If you are not in the directory that is housing your json file, you will need to provide the full path when you run the command later.

1. Run the mongoimport command, change the flags accordingly.

   **Flag legend**
   
   --uri : Put in your mongodb URI string after this flag
   
   -c : The name of your collection

   The command should have the following structure:

   `mongoimport --uri <MongoDB URI string with extra appenddage> -c <collection name> --jsonArray --type=json <filename or full filepath>`

   Here is an example command for reference:
   
   `mongoimport --uri "mongodb://mongo:zafDXoaiuTw80q3bMvqM@containers-us-west-207.railway.app:7018/bgg?authSource=admin" -ccomment --jsonArray --type=json comment.json`

You are technically done. What's left is to generate a domain and cross your fingers

   Project service > Settings > under domain, Generate domain
   
---
## Some possible fixes to errors you may encounter
1. `railway up` is not working / Build is failing

   Make sure your project's java version is one that railway supports (e.g. 19, 17, or look at the definitive list <a href="https://nixpacks.com/docs/providers/java">here</a>)
   You can confirm that by looking at the version in your pom.xml file located in the root of your project, between <java.version> tags.

   If it is a version that is not supported, change it to either 19 or 17.

   You would have to add another environment variable to your project service on railway if you changed the version to 19. Follow the steps in point 4 of **Provision and link a mongodb instance** to change it, but use these values:

   VARIABLE_NAME: `NIXPACKS_JDK_VERSION`
   
   VALUE: `19`

2. Application is not able to retrieve data from MongoDB.

  Double check that your environment variable is properly set, and have the following structure:
  
  `<mongodb URI>/<database name>?authSource=admin`

  Example:
  
  mongodb://mongo:zafDXoaiuTw80q3bMvqM@containers-us-west-207.railway.app:7018/boardgames?authSource=admin

# PAF Commands
## Mongo CLI commands:

### Mongoimport locally

`mongoimport -hlocalhost -p27017 -d<databasename> -c<collectionname> --jsonArray --type=json <filename or filepath>`

Example:

`mongoimport -hlocalhost -p27017 -dshows -ctv-show --jsonArray --type=json tv-shows.json`

### Mongoimport to remote

`mongoimport --uri <MongoDB URI string>/<database name>?authSource=admin -c <collection name> --jsonArray --type=json <filename or full filepath>`

Example:
   
`mongoimport --uri "mongodb://mongo:zafDXoaiuTw80q3bMvqM@containers-us-west-207.railway.app:7018/bgg?authSource=admin" -ccomment --jsonArray --type=json comment.json`
