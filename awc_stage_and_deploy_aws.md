**Staging**: When setup, whenever a commit is pushed to the repo's **stage** branch, AWCbits will launch a new version of the site from the stage branch. You can also manually stage the site by going to the project's Sparrow page and click Stage.

**Deploying**: Push the codebase from the **master** branch to the live webserver.

#Setup
## Repo
Be sure to have both **master** and **stage** branches setup in the project's repo.  

##Sparrow
In Sparrow, go to the project page that this repo is associated with.  Click Update Project.

Under Deployment: 
**Host URL/IP:** IP address of the live server.  Needed to deploy, not stage.  
**Host Username:** Username of the client on the live server.  Needed to deploy, not stage.  
**Bitbucket URL:** Last Uri of the repo url  
**Site Location on Host:** Web directory of the live site  Needed to deploy, not stage.   
**Excluded Files:** Files not to be includes in the stage/deploy.  These will be any structure config files included in the repo but not required for stage/production such as git, grunt, etc, as well as any S3 mounts, cache and log directories.  
**Post Stage Commands:** Commands ran on AWCbits after stage is done with its sync.  The would be copy commands to move over any config files not in version control.  
**Post Deploy Commands:** Commands ran on the live server after deployment is done with its sync.

Once complete, click Update.  Then click on Update Project again, you'll find a POST url at the bottom of the page.  This needs to be added to the Bitbucket repo.  On Bitbucket's website, find the repo, click on Settings->Hooks.  Select the POST hook, click Add hook, then paste the hook in URL.

##Staging
AWCbits will have a ~/system/new_site_apache.sh script.  When ran, it will ask if you'd like to setup staging and deploying. Say 'Y' to staging.

Now, when you make a commit/push to stage, or press **Stage** on the project page, deploy will stage the site to AWCbits.

##Deploying
To setup Deploying, you need to have a live EC2 instance running with the client and site already setup. You can either setup Deploying when creating the site on AWCbits, or you can run ~/system/client_deploy_stage.sh.

You should be able to Deploy from Sparrow.  Just remember to checkout master and merge any changes from Stage to Master beforehand.												