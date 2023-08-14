This guide will show you how to deploy a Grails application to Heroku and bind it to the Postgres database service.

## Prerequisites

* Java, Maven, Git, and the Heroku client (as described in the [basic Java quickstart](/java))
* An installed version of [Postgres](http://www.postgresql.org/) to test locally
* An installed version of [Grails](http://grails.org/Installation) 1.3.7 or 2.x (this guide assumes 2.2.0)

## Create a Grails app

If you don't already have a Grails app, you can create one quickly following the standard [Grails Quick Start](http://grails.org/Quick+Start) or clone this sample from github:

    :::term
    $ git clone git://github.com/heroku/devcenter-grails.git
    
## Set up the database

Heroku automatically provisions a small database when you create a Grails application and sets the `DATABASE_URL` environment variable to a URL of the format

    postgres://user:password@hostname/path

You can also provision a larger database service yourself using the `heroku addons` command. Either way, the database connection information will be stored in the `DATABASE_URL` variable.

Configure your app to use this database by changing the `production` database configuration in `grails-app/conf/DataSource.groovy` to this:

    :::groovy
    production {
        dataSource {
            dbCreate = "update"
            driverClassName = "org.postgresql.Driver"
            dialect = org.hibernate.dialect.PostgreSQLDialect
        
            uri = new URI(System.env.DATABASE_URL?:"postgres://test:test@localhost/test")

            url = "jdbc:postgresql://"+uri.host+uri.path
            username = uri.userInfo.split(":")[0]
            password = uri.userInfo.split(":")[1]
        }
    }

## Configure dependencies

Edit `grails-app/conf/BuildConfig.groovy` to add the postgres JDBC driver as a dependency. Your `BuildConfig.groovy`'s `dependencies` section should look like this:

    :::groovy
    dependencies {
        runtime 'postgresql:postgresql:8.4-702.jdbc3
    }


## Check in to Git
Create a `.gitignore` file [tailored for Grails](https://github.com/github/gitignore/blob/master/Grails.gitignore) to prevent generated files from being checked in:

    *.iws
    *Db.properties
    *Db.script
    .settings
    .classpath
    .project
    eclipse
    stacktrace.log
    target
    /plugins
    /web-app/plugins
    /web-app/WEB-INF/classes
    web-app/WEB-INF/tld/c.tld
    web-app/WEB-INF/tld/fmt.tld

Check your app into Git:

    :::term
    $ git init
    $ git add .
    $ git commit -m init

## Create a Heroku app and deploy

Create the app on the Cedar stack

    :::term
    $ heroku create

### Optional: Declare Process Types With Procfile

You declare how you want your application executed in `Procfile` in the project root. Create this file with a single line:

    :::term
    web: java $JAVA_OPTS -jar server/jetty-runner.jar --port $PORT target/*.war

Running Grails on the Cedar stack automatically generates the Procfile above. But you can take full control of how your app is executed by defining your own Procfile.

## Deploy your code:

    :::term
    $ git push heroku master
    Counting objects: 110, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (80/80), done.
    Writing objects: 100% (110/110), 163.34 KiB, done.
    Total 110 (delta 17), reused 110 (delta 17)
    -----> Heroku receiving push
    -----> Fetching custom buildpack... done
    -----> Grails app detected
    -----> Grails 2.2.0 app detected
    -----> Installing Grails 2.2.0.....
    -----> Done
    -----> Executing grails -plain-output -Divy.default.ivy.user.dir=/app/tmp/repo.git/.cache war
       
           |Loading Grails 2.2.0
           |Configuring classpath
           |Downloading: ivy-1.0.1.RELEASE.xml
           ...
           |Done creating WAR target/grails20Test-0.1.war
    -----> No server directory found. Adding jetty-runner 7.5.4.v20111024 automatically.
    -----> Discovering process types
           Procfile declares types  -> (none)
           Default types for Grails -> web
    -----> Compiled slug size is 31.1MB
    -----> Launching... done, v6
           http://young-summer-1535.herokuapp.com deployed to Heroku

Congratulations! Your Grails app is now up and running on Heroku. Open it in your browser with:

    :::term  
    $ heroku open

## Local testing

Normally you run your Grails apps locally with `grails run-app`. This works great for quick, iterative development, and is fine to use when developing for Heroku. However, it is often useful to run the app using the same configuration as the server. 

### Add an embedded server

As you might have noticed from the build output above. Heroku added Jetty as the web server to run your app, but you don't have to let Heroku do this. Simply download the jetty-runner jar yourself and add it to your project:

    $ mkdir server
    $ curl http://repo2.maven.org/maven2/org/mortbay/jetty/jetty-runner/7.5.4.v20111024/jetty-runner-7.5.4.v20111024.jar > server/jetty-runner.jar

You can put any server configuration in the server directory. For example, you can put a complete Tomcat distribution there.

### Configure your local database URL

To run your app in the same configuration as Heroku, you must run a local Postgres database. It's a common source of issues to use different databases in development and production. Set up a database on your Postgres instance and set the DATABASE_URL variable in your environment:

    $ export DATABASE_URL=postgres://user:pass@localhost/dbname

### Run Your App

<div class="callout" markdown="1">
Note: you can also run your app with foreman using `foreman start` if you specified a Procfile previously. [Read more about foreman and procfiles](http://devcenter.heroku.com/articles/procfile).
</div>

Build the war file:

    $ grails war

Execute jetty runner:

    $ java -jar server/jetty-runner.jar target/*.war

    2012-01-23 15:22:30.792:INFO:omjr.Runner:Runner
    2012-01-23 15:22:30.793:WARN:omjr.Runner:No tx manager found
    2012-01-23 15:22:30.820:INFO:omjr.Runner:Deploying file:/Users/test/dev/devcenter-grails/target/grails20Test-0.1.war @ /[o.e.j.w.WebAppContext{/,null},file:/Users/test/dev/devcenter-grails/target/grails20Test-0.1.war]
    2012-01-23 15:22:30.836:INFO:oejs.Server:jetty-7.x.y-SNAPSHOT
    2012-01-23 15:22:30.865:INFO:oejw.WebInfConfiguration:Extract jar:file:/Users/test/dev/devcenter-grails/target/grails20Test-0.1.war!/ to /private/var/folders/b2/wfscp_952tn6gd6dhssbr7q00000gn/T/jetty-0.0.0.0-8080-grails20Test-0.1.war-_-any-/webapp
    2012-01-23 15:22:32.683:INFO:oejpw.PlusConfiguration:No Transaction manager found - if your webapp requires one, please configure one.
    2012-01-23 15:22:33.478:INFO:/:Initializing Spring root WebApplicationContext
    2012-01-23 15:22:37.780:INFO:oejsh.ContextHandler:started o.e.j.w.WebAppContext{/,file:/private/var/folders/b2/wfscp_952tn6gd6dhssbr7q00000gn/T/jetty-0.0.0.0-8080-grails20Test-0.1.war-_-any-/webapp/},file:/Users/test/dev/devcenter-grails/target/grails20Test-0.1.war
    2012-01-23 15:22:37.849:INFO:/:Initializing Spring FrameworkServlet 'grails'
    2012-01-23 15:22:37.870:INFO:oejs.AbstractConnector:Started SelectChannelConnector@0.0.0.0:8080 STARTING

Your app is now up on <http://localhost:8080>.

Of course you don't have to use Jetty at all. As you have seen, you are in full control of what web server you use for your app and how it is launched by Heroku. This gives you the benefit of being able to control your production runtime in detail and being able to run exactly the same configuration in development and production.

## Grails Heroku plugin

The [Grails Heroku plugin](http://grails.org/plugin/heroku) provides a set of simple commands to set up your Grails app with Heroku add-on services like [Postgres](https://addons.heroku.com/heroku-postgresql), [Memcached](https://addons.heroku.com/memcache), [Redis](https://addons.heroku.com/redistogo), MongoDB from [MongoLabs](https://addons.heroku.com/mongolab) or [MongoHQ](https://addons.heroku.com/mongohq) and [RabbitMQ](https://addons.heroku.com/rabbitmq).
