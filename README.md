This guide will show you how to deploy a Grails application to Heroku and bind it to the free Postgres database service.

## Prerequisites

* Java, Maven, Git, the Heroku client, and Foreman (as described in the [basic Java quickstart](/java))
* An installed version of [Postgres](http://www.postgresql.org/) to test locally
* An installed version of [Grails](http://grails.org/Installation) 1.3.7 or 2.0.0 (this guide assumes 2.0.0)

## Create a Grails app

If you don't already have a Grails app, you can create one quickly following the standard [Grails Quick Start](http://grails.org/Quick+Start) or clone this sample from github:

    $ git clone git://github.com/heroku/devcenter-grails.git
    
## Set up the database

Heroku automatically provisions a small database when you create a Grails application and sets the `DATABASE_URL` environment variable to a URL of the format

    postgres://user:password@hostname/path

You can also provision a larger database service yourself using the `heroku addons` command. Either way, the database connection information will be stored in the `DATABASE_URL` variable.

Configure your app to use this database by changing the `production` database configuration in `grails-app/conf/DataSource.groovy` to this:

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

## Configure Dependencies

Edit `grails-app/conf/BuildConfig.groovy` to add the postgres JDBC driver as a dependency and to enable Maven Central and the local Maven cache as repositories. Your `BuildConfig.groovy` should look like this:

    grails.project.class.dir = "target/classes"
    grails.project.test.class.dir = "target/test-classes"
    grails.project.test.reports.dir = "target/test-reports"
    grails.project.target.level = 1.6
    grails.project.source.level = 1.6
    //grails.project.war.file = "target/${appName}-${appVersion}.war"
    grails.project.dependency.resolution = {
        // inherit Grails' default dependencies
        inherits("global") {
            // uncomment to disable ehcache
            // excludes 'ehcache'
        }
        log "warn" // log level of Ivy resolver, either 'error', 'warn', 'info', 'debug' or 'verbose'
        checksums true // Whether to verify checksums on resolve
        repositories {
            inherits true // Whether to inherit repository definitions from plugins
            grailsPlugins()
            grailsHome()
            grailsCentral()
            mavenCentral()

            // uncomment the below to enable remote dependency resolution
            // from public Maven repositories
            mavenLocal()
            //mavenCentral()
            //mavenRepo "http://snapshots.repository.codehaus.org"
            //mavenRepo "http://repository.codehaus.org"
            //mavenRepo "http://download.java.net/maven/2/"
            //mavenRepo "http://repository.jboss.com/maven2/"
        }
        dependencies {
            // specify dependencies here under either 'build', 'compile', 'runtime', 'test' or 'provided' scopes eg.

            // runtime 'mysql:mysql-connector-java:5.1.13'
            runtime 'postgresql:postgresql:8.4-702.jdbc3'
        }
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

    $ git init
    $ git add .
    $ git commit -m init

## Create a Heroku app and deploy

Create the app on the Cedar stack

    $ heroku create --stack cedar

Deploy your code:

    $ git push heroku master
    Counting objects: 110, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (80/80), done.
    Writing objects: 100% (110/110), 163.34 KiB, done.
    Total 110 (delta 17), reused 110 (delta 17)
    -----> Heroku receiving push
    -----> Fetching custom buildpack... done
    -----> Grails app detected
    -----> Grails 2.0.0 app detected
    -----> Installing Grails 2.0.0.....
    -----> Done
    -----> Executing grails -plain-output -Divy.default.ivy.user.dir=/app/tmp/repo.git/.cache war
       
           |Loading Grails 2.0.0
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

Normally you run your Grails apps locally with `grails run-app`. This works great for quick, iterative development. But sometimes, it is useful to run the app in the same configuration as the server. 

### Add an embedded server

As you might have noticed from the build output above. Heroku added Jetty as the web server to run your app, but you don't have to let Heroku do this. Simply download the jetty-runner jar yourself and add it to your project:

    $ mkdir server
    $ curl http://repo2.maven.org/maven2/org/mortbay/jetty/jetty-runner/7.5.4.v20111024/jetty-runner-7.5.4.v20111024.jar > server/jetty-runner.jar

You can put any server configuration in the server directory. For example, you can put a complete Tomcat distribution there.

### Declare Process Types With Procfile

You declare how you want your application executed in `Procfile` in the project root. Create this file with a single line:

    :::term
    web: java $JAVA_OPTS -jar server/jetty-runner.jar --port $PORT target/*.war

When you deployed to Heroku before, this process type was declared by default. But you can take full control of how your app is executed by defining your own Procfile.

You can now run your app locally using the same configuration as Heroku.

### Configure your local database URL

To run your app in the same configuration as Heroku, you must run a local Postgres database. It's a common source of issues to use different databases in development and production. Set up a database on your Postgres instance and set the DATABASE_URL variable in your environment:

    $ export DATABASE_URL=postgres://user:pass@localhost/dbname

### Run Your App with Foreman

First build the war file:

    $ grails war

Then run your app with Foreman:

    $ foreman start
    17:10:59 web.1     | started with pid 37303
    17:11:00 web.1     | 2011-09-08 17:11:00.207:INFO:omjr.Runner:Runner
    17:11:00 web.1     | 2011-09-08 17:11:00.207:WARN:omjr.Runner:No tx manager found
    17:11:00 web.1     | 2011-09-08 17:11:00.247:INFO:omjr.Runner:Deploying file:/Users/jjoergensen/dev/grails-sample/target/grails-sample-0.1.war @ /
    17:11:00 web.1     | [o.e.j.w.WebAppContext{/,null},file:/Users/jjoergensen/dev/grails-sample/target/grails-sample-0.1.war]
    17:11:00 web.1     | 2011-09-08 17:11:00.265:INFO:oejs.Server:jetty-7.x.y-SNAPSHOT
    17:11:00 web.1     | 2011-09-08 17:11:00.300:INFO:oejw.WebInfConfiguration:Extract jar:file:/Users/jjoergensen/dev/grails-sample/target/grails-sample-0.1.war!/ to /private/var/folders/-G/-Gcbk9Y+FruX6u1ylpMsM+Ie7w+/-Tmp-/jetty-0.0.0.0-5000-grails-sample-0.1.war-_-any-/webapp
    17:11:03 web.1     | 2011-09-08 17:11:03.122:INFO:oejpw.PlusConfiguration:No Transaction manager found - if your webapp requires one, please configure one.
    17:11:04 web.1     | 2011-09-08 17:11:04.363:INFO:/:Initializing Spring root WebApplicationContext
    17:11:11 web.1     | 2011-09-08 17:11:11.106:INFO:oejsh.ContextHandler:started o.e.j.w.WebAppContext{/,file:/private/var/folders/-G/-Gcbk9Y+FruX6u1ylpMsM+Ie7w+/-Tmp-/jetty-0.0.0.0-5000-grails-sample-0.1.war-_-any-/webapp/},file:/Users/jjoergensen/dev/grails-sample/target/grails-sample-0.1.war
    17:11:11 web.1     | 2011-09-08 17:11:11.188:INFO:/:Initializing Spring FrameworkServlet 'grails'
    17:11:11 web.1     | 2011-09-08 17:11:11.221:INFO:oejs.AbstractConnector:Started SelectChannelConnector@0.0.0.0:5000 STARTING

Your app is now up on <http://localhost:5000>.

You can check in these changes and push them to Heroku:

    $ git add .
    $ git commit -m "added Procfile and jetty-runner"
    $ git push heroku master
    ...

Of course you don't have to use Jetty at all. As you have seen, you are in full control of what web server you use for your app and how it is launched by Heroku. This gives you the benefit of being able to control your production runtime in detail and being able to run exactly the same configuration in development and production.


