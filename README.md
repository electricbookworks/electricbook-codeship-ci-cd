# Test and deploy a Jekyll site with CodeShip Basic

This is a workflow for deploying Jekyll sites with CodeShip Basic. It follows good practices by building once, and deploying a tested build to staging and live webservers based on Git branches and tags. It also allows deploying previews, for instance for building a specific feature branch.

- [Setup](#setup)
- [Deploying previews](#deploying-previews)
- [Deploying to staging and live](#deploying-to-staging-and-live)

## Setup

1. **SSH access:** If you want to use SSH, for the fastest deployments, ensure you have SSH access to your preview, staging and live webservers. On some web hosts, SSH access must be requested and enabled by support. You may still need to do this for your domain. Otherwise, deployment will use FTP.
2. **Add the scripts:** Make sure your repo includes the `test.sh` and `deploy.sh` scripts here. They should be in the repo's root folder. In addition, our scripts assume that your repo includes an up-to-date `_configs/_config.live.yml` file, containing any configuration settings for live builds.
3. **Add a CodeShip project:** Add a project to CodeShip and connect your repo. Choose 'CodeShip Basic' for your project type, not 'Pro'.
4. **Tests:** In the CodeShip project settings, go to Tests. Choose 'Ruby' from the list of technology defaults. Note: if you are setting up a site that does not build with Jekyll, such as an Electric Book remote-media repo, you can leave the Setup Commands blank.

   1. Under 'Setup commands' you *may* need to include commands that set up the environment (i.e. the machine that builds your site) correctly. By default, CodeShip will include `bundle install` here. For instance, if your site's `Gemfile.lock` has been generated with Bundler 2 (see the end of the `Gemfile.lock` to check), you need to install Bundler 2. To do this, add these lines before `bundle install`:

      ``` sh
      # Uninstall any local and global bundler installations
      gem uninstall -x -a bundler
      rvm @global do gem uninstall -x -a bundler

      # Install newer version of Bundler
      gem install bundler -v 2.2.19
      ```

    You can change the version number there as appropriate for your project.

   2. Then under 'Configure Test Pipelines', click 'Add Pipeline' and add a tab called 'Test Commands'. In that tab, add these lines:

      ``` sh
      # Give permission for the script to run
      chmod +x test.sh

      # Run the test script
      ./test.sh
      ```

   Note that the test script here does not actually include any tests. You'll need to add those based on your project needs.

5. **Deploy:** In Deploy settings, create deploy settings for three 'branches':

   - create one deployment for the `live`
   - one for branches that 'start with' `preview`
   - one for branches that 'start with' `release`.

   For each one, choose Script Deployment and add these deploy commands:

   ``` sh
   # Give the script permission to run
   chmod +x deploy.sh

    # Run the deploy script
   ./deploy.sh
   ```

   By default, this will build and deploy to the staging server any further changes committed/merged to the live branch.

6. **Build triggers:** Next, set the Build Triggers. You want to run builds on the branch you're deploying (e.g. `live`), and on `^preview.*$` and  `^release.*$`, which is any branch *or tag* that starts with 'preview' and 'release', respectively. (For Git purposes, a tag is a branch, but without any further commits). To do this, select 'Run builds for these branches only', and add these three lines to the 'Enter one branch per line' box:

   ``` sh
   live
   ^preview.*$
   ^release.*$
   ```

   For the other settings:

   - set to run only on commits, merges and tags
   - set auto-supersede builds off (we want staging builds to complete before deploying to live).

7. **Environment variables:** In CodeShip project settings, you need to add several environment variables. These set exactly how you'll deploy and where to.

| Key                      | Example value                    | Alt. example  | Notes                                                                                                           |
| ------------------------ | -------------------------------- | ------------- | --------------------------------------------------------------------------------------------------------------- |
| DEPLOY_METHOD_LIVE       | FTP                              | SSH           | If you don't set this, FTP is default. SSH is much faster, but requires high-level server access.               |
| DEPLOY_METHOD_STAGING    | FTP                              | SSH           | If you don't set this, FTP is default. SSH is much faster, but requires high-level server access.               |
| DEPLOY_METHOD_PREVIEWS   | FTP                              | SSH           | If you don't set this, FTP is default. SSH is much faster, but requires high-level server access.               |
| DESTINATIONPATH_STAGING  | public_html/mybooks              |               | Required. If your SSH user connects directly to the destination folder, leave this value blank or don't set it. |
| DESTINATIONPATH_LIVE     | public_html/mybooks              |               | Required. If your SSH user connects directly to the destination folder, leave this value blank or don't set it. |
| DESTINATIONPATH_PREVIEWS | public_html                      |               | Required. If your SSH user connects directly to the destination folder, leave this value blank or don't set it. |
| SSH_STAGING              | staginguser@staging.example.com  |               | Not required if using FTP.                                                                                      |
| SSH_LIVE                 | liveuser@example.com             |               | Not required if using FTP.                                                                                      |
| SSH_PREVIEWS             | previewuser@previews.example.com |               | Not required if using FTP.                                                                                      |
| SSH_STAGING_PORT         | 2222                             |               | Only required if your staging-site host uses a non-standard SSH port (i.e. not port 22)                         |
| SSH_LIVE_PORT            | 2222                             |               | Only required if your live-site host uses a non-standard SSH port (i.e. not port 22)                            |
| SSH_PREVIEWS_PORT        | 2222                             |               | Only required if your previews site host uses a non-standard SSH port (i.e. not port 22)                        |
| FTP_HOST_LIVE            | example.com                      | 42.653.343.33 | Not required if using SSH.                                                                                      |
| FTP_USER_LIVE            | dkufh7fhrf                       | jo@mysite     | Not required if using SSH.                                                                                      |
| FTP_PASSWORD_LIVE        | kfuh4i87f3ufhr7f743hf            |               | Not required if using SSH.                                                                                      |
| FTP_HOST_PREVIEWS        | previews.example.com             |               | Not required if using SSH.                                                                                      |
| FTP_USER_PREVIEWS        | hfiuhf7f                         |               | Not required if using SSH.                                                                                      |
| FTP_PASSWORD_PREVIEWS    | ufhwfh8374fi4hf4fwuefiw          |               | Not required if using SSH.                                                                                      |
| FTP_HOST_STAGING         | staging.example.com              |               | Not required if using SSH.                                                                                      |
| FTP_USER_STAGING         | rufhi4ff                         |               | Not required if using SSH.                                                                                      |
| FTP_PASSWORD_STAGING     | fiufuhr87ffeoifiw7f              |               | Not required if using SSH.                                                                                      |

8. **SSH key:** If you are using SSH for deployments, get the project's CodeShip public key from the 'General' tab in project settings. Add this CodeShip key to both your staging and your live servers. On many webservers, you'll add this key as a line in the `~/.ssh/authorized_keys` file on the server.

9. **Done!** At this point:

   - Any commit to the `live` branch will be automatically deployed to staging.
   - Any commit on any branch with a tag that starts with `preview` will be deployed to staging, in a directory named after the tag.
   - Any commit on any branch with a tag that starts with `release` will be deployed to live. **Never add a `release` tag to any branch other than `live`.** The scripts should protect you from releasing a non-live-branch build to live, but it's best not to take this chance.

## Deploying previews

To deploy a preview to the staging server, you'll use git tags that start with `preview`. To do this, first make sure that your current local Git repo is in the state you want to preview. This process will build a preview of the most recent commit. Then:

- If you are working in the original upstream repo (don't type the `#` comment lines):

   ``` sh
   # Add a preview-* tag to the last commit
   # You need the -f (aka --force) if you're using a tag you've used before.
   # E.g. if you're updating the `more-potatoes` preview to a later commit.
   git tag -a preview-more-potatoes -f -m "A short description"

   # Push the tag.
   # Again, the -f forces git to update an existing tag on the remote.
   git push preview-more-potatoes -f
   ```

- If you are working in a fork (don't type the `#` comment lines):

   ``` sh
   # Make sure you have the latest version of the relevant branch from upstream
   # For most previews the relevant branch to pull from will be 'master' not 'more-potatoes'
   git pull upstream more-potatoes

   # Add a preview-* tag to the last commit
   # You need the -f (aka --force) if you're using a tag you've used before.
   # E.g. if you're updating the `more-potatoes` preview to a later commit.
   git tag -a preview-more-potatoes -f -m "A short description"

   # Push the tag.
   # Again, the -f forces git to update an existing tag on the remote.
   git push upstream preview-more-potatoes -f
   ```

CodeShip will spot your tag, build the site and run the tests, and then deploy the site to your staging server in a directory named for your tag, e.g. `example.com/preview-more-potatoes/`.

**Technical note:** We're using annotated tags, not lightweight tags, hence the `-a` and the `-m` followed by a short message. This is because by default CodeShop does a shallow clone to reduce the time taken to fetch the repo, and shallow clones do not include lightweight tags.

## Deploying to staging and live

When you merge a PR into the `live` branch, CodeShip will deploy to the staging server automatically. If the staging website looks fine and is approved, you are ready to deploy to live.

To do this, you will tag the latest commit on the `live` branch with a tag that starts with the word 'release', e.g. `release-v1.8.6`. When you push that tag, CodeShip will deploy the latest build to the live webserver. The commands you use for this depend on whether you are working in the original, upstream repo, or your fork of it.

- If you are working in the original upstream repo (don't type the `#` comment lines):

   ``` sh
   # Checkout the live branch
   git checkout live

   # Get the latest changes
   git pull

   # Add a release-* tag to the last commit
   git tag -a release-v1.8.5 -f -m "A short description"

   # Push that tag
   git push --tags

   # Checkout master again for safety
   git checkout master
   ```

- If you are working in a fork (don't type the `#` comment lines):

   ``` sh
   # Make a new live branch if you don't yet have one...
   git checkout -b live

   # ...or just checkout if you already have a live branch
   git checkout live

   # Pull the upstream's live branch into your live branch
   git pull upstream live

   # Add a release-* tag to the last commit
   git tag -a release-v1.8.5 -f -m "A short description"

   # Push that tag to upstream's live branch
   git push upstream --tags

   # Checkout master again for safety
   git checkout master
   ```

Note that this deploys only the last build, and does not build and test again. This is to ensure that you deploy the exact same build that was deployed to staging. When CodeShip runs the test and deploy scripts, the scripts check the tag on the last commit. If it starts with 'release', they deploy the most recent build to the live server. If it doesn't, it builds and deploys to staging.
