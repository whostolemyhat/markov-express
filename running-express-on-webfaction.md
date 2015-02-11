running express on webfaction

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
10. Run your app using forever: `forever start app/bin/www`
11. Visit your site - should be working! 502 Bad Gateway means you are using the wrong port or the server isn't running. You can now update the app's files by changing locally, commiting to git then pushing to web.