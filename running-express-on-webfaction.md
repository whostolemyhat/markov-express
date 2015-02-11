running express on webfaction

Install node:
https://gist.github.com/marcoslhc/3909289

1. Log into dashboard
2. Create Node app. Make a note of the port it's running on.
4. On local machine, install Express generator `npm install -g express-generator` `express app`
5. Edit app/bin/www to have the port from Webfaction
6. Create git repo and add all files `git init; git add -A; git commit -m "basic Express setup"`
3. Create Git app on webfaction. Follow instructions here to get a post-receive hook. (my structure ended up being /home/user/webapps/nodeapp/app)
http://docs.webfaction.com/software/git.html#pushing-and-pulling-with-a-repository https://community.webfaction.com/questions/1246/using-git-with-applications http://toroid.org/ams/git-website-howto
7. SSH into server. cd to your node app. edit bin/start to point to your files (/app).
8. Follow these steps to make sure NPM is aliased in the path: http://docs.webfaction.com/software/nodejs.html#installing-packages-with-npm
9. Install forever `npm install -g forever`
10. Run your app using forever: `forever start -a -o logs/out.log -e logs/error.log --uid "production" app/bin/www`
    uid = id. can use to stop ie `forever stop production`
    API here: https://github.com/foreverjs/forever
    modify the nodeapp/bin/start and stop files to replace them with forever commands
    update your post-receive file to run the bin/start and stop commands to refresh your app
    http://blog.tyrsius.com/webfaction-git-deploy/
11. Visit your site - should be working! 502 Bad Gateway means you are using the wrong port or the server isn't running. You can now update the app's files by changing locally, commiting to git then pushing to web.

Deploying code and sites via Git is really useful since it ensures all your changes are tracked in source control
and means you can update your entire site with a single command in the terminal - so much easier than FTP! If you've ever created an app on Heroku, you'll be familiar with deploying code using Git,
and it's (fairly) straightforward to set up on Webfaction. As an example, I'll show you how to set up Git deployment on your Webfaction account and then we'll deploy a basic Express site
running on NodeJS using Git.

Part 1: Install NodeJS

The first step is to install NodeJS on Webfaction; the simplest way is to [follow the instructions here](https://gist.github.com/marcoslhc/3909289), as taken from various answers 
on the Webfaction knowledge base. I've replicated the relevant parts below. You'll need to SSH into your Webfaction account, change to the $HOME directory then run the following:
    
    # from https://gist.github.com/marcoslhc/3909289
    mkdir -p $HOME/src
    cd $HOME/src
    wget 'http://nodejs.org/dist/v0.12.0/node-v0.12.0.tar.gz' # check the NodeJS site for the latest version
    tar -xzf node-v0.12.0.tar.gz
    cd node-v0.12.0
     
    # All of these scripts use "#!/usr/bin/env python". Let's make that mean python2.7:
    PATH_BACKUP="$PATH"
    mkdir MYPY
    ln -s $(which python2.7) $PWD/MYPY/python
    export PATH="$PWD/MYPY:$PATH"
     
    ./configure --prefix=$HOME
    make         # 5.5m
    make install
     
    # restore the PATH (we don't need `$PWD/MYPY/python` anymore).
    # Then, make sure $HOME/bin is on the PATH
    PATH="$PATH_BACKUP"
    export PATH=$HOME/bin:$PATH
    echo 'export PATH="$HOME/bin:$PATH"' >> $HOME/.bashrc
    hash -r
     
    # Set the node modules path
    export NODE_PATH="$HOME/lib/node_modules:$NODE_PATH"
    echo 'export NODE_PATH="$HOME/lib/node_modules:$NODE_PATH"' >> $HOME/.bashrc
     
    # now install forever module - we'll need this later
    cd $HOME
    npm install -g forever

Run `node -v` to make sure node is installed correctly (should output v0.12.0).

Part 2: Set up Applications

The next step is to set up two applications in your Webfaction dashboard; the first is a Git application which will handle the repos and hooks, and the other is the Node application which
will serve the Express app.

Create a new NodeJS app in Webfaction, and make a note of the port the application is running on since we'll need to use this later. 
Next, set up a Git application: [follow the instructions here](http://docs.webfaction.com/software/git.html) up to and including the 'Creating a New Repository' section. After creating
a Git application, we need to create and edit the post-receive hook to update our files after we've pushed changes to Webfaction. [Git hooks](http://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)
 contain commands which Git will run automatically when you push code to the repo, and we'll use the post-receive hook to move the code to the correct place then restart the app.

[Follow these instructions to set up a post-receive hook](http://toroid.org/ams/git-website-howto), and make sure you're using the correct file 
paths [as described in the answer here](https://community.webfaction.com/questions/1246/using-git-with-applications). The post-receive hook should be in 
`/home/username/webapps/yourgitapp/repos/appname.git/hooks/post-receive` and should look something like this:
    
    #!/bin/sh
    GIT_WORK_TREE=/home/username/webapps/django/myproject git checkout -f master
    GIT_WORK_TREE=/home/username/webapps/django/myproject git reset --hard

Webfaction automatically created a pair of scripts in your NodeJS app when you set it up - these can be found in webapps/nodeapp/bin and are called `start` and `stop`.
We'll edit these to use the `forever` module to keep Express running (which is why we installed `forever` earlier). Open webapps/nodeapp/bin/start (using nano or vi or similar) and edit it to
look like the following:

On your local machine (which I'm assuming already has Node and NPM installed), install the Express generator:

`npm install -g express-generator`

and then run `express app` to create a bare-bones Express application. Run `node bin/www` to check it's working ([localhost:3000](http://localhost:3000) by default).
Edit /bin/www to have the port used by the NodeJS application in Webfaction, then run the following to create and commit all files to Git:

    git init
    git add -A
    git commit -m "Basic bare-bones Express app"