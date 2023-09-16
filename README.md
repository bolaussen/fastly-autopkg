# Using AutoPKG with GitHub Actions

This repo contains a forked and modified example of the files and configurations needed to run AutoPKG using GitHub Actions and uploading to Jamf Pro.

I have supplied an example override file for Google Chrome. 

The example ``` autopkg.yml ``` file is set to run on an automated schedule, Monday - Friday at 9am (using Cron time). This workflow file also does the following:
1. Checks out the repo where the workflow file exists using the Checkout Action from the GitHub Actions Marketplace
2. Installs the stated version of Python using the Setup Python Action from the GitHub Actions Marketplace
3. Installs the Python dependencies
4. Installs AutoPkg
5. Configures AutoPkg and Git
6. Uses the autopkg_tools.py script to process each override recipe and creates a pull request if trust info is missing or has changed
7. Posts build results to designated Slack channel if integration is set up.

> The ``` autopkg_tools.py ``` script posts build results to Slack but only trust info changes or if
> the run
> has failed. If you would like to see when recipes are processed successfully, add Graham Pugh's
> [JamfUploaderSlacker processor](https://github.com/grahampugh/jamf-upload/blob/main/JamfUploaderProcessors/READMEs/JamfUploaderSlacker.md) to the end of each of your recipes (view the provided override file for
> an example). 

# How to Use This Repository

1. Create a new repo with Actions enabled in GitHub.
2. Create a .github/workflows directory in GitHub if one doesn’t already exist.
    1. To view this directory using macOS Finder, make sure to enable viewing hidden files           (CMD+Shift+period).
3. Clone or copy the autopkg.yml workflow file to the .github/workflows directory in your repository.
    1. Evaluate this workflow file and decide if you want it to run on an automated schedule or some other way.
4. Create the rest of the files and directories within GitHub or your locally cloned repository.
    1. overrides directory
    2. autopkg_tools.py
    3. recipe_list.json
    4. repo_list.txt
    5. requirements.txt
5. Configure your GitHub Secrets environment with credentials for your Jamf environment, preferably a Jamf account that is dedicated to AutoPkg as well as your Slack Webhook URL (if you want to set up the Slack integration).
    1. If you use JamfUploader, consider which processors you will be using and grant your Jamf account  [the correct permissions](https://github.com/grahampugh/jamf-upload/wiki/JamfUploader-AutoPkg-Processors#jamf-account-privileges) for the actions you will be taking.
6. Create an override recipe and any associated templates and place them in the /overrides directory.
7. Add the recipe to recipe_list.json.
8. Add the full URL of the parent repository to repo_list.txt.
9. Try running your first recipe!
    1. Click on the Actions tab within your repository
    2. Click on your workflow file on the left-hand side
    3. Click the Run Workflow button
    4. Either enter the path to a single recipe or click the *Run Workflow* button to run all the recipes in your recipe_list.json file

# How to Create a Slack App

If you have admin privileges in Slack, the following instructions should work for you, if you don’t then submit a request to your Slack administrator for help creating the application.
> 

1. Navigate to [https://api.slack.com/apps](https://api.slack.com/apps) using your admin profile.
2. Click the green *Create New App* button.
3. Give the app a name and choose the workspace you would like to install the app.
4. Once you can see the *Settings* and *Features* menu of your new app, click on *OAuth & Permissions* underneath the *Features* heading.
5. Scroll down to *Scopes* and click *Add an OAuth Scope*.
6. In the drop-down list, scroll down until you see the option for *incoming-webhook* and select it.
7. Once *incoming-webhook* is added, scroll back up to *OAuth Tokens for Your Workspace* and click *Install to Workspace*.
8. You will be prompted to select which Slack channel you would like to connect your new Slack app to and click *Allow*.
    1. The Slack channel, either public or private, needs to be created before step 8 so that it will show up in this list.
9. You will need the following piece of information to add as a secret within your GitHub repository
    1. Webhook URL which can be found under *Settings → Install App*.
  
# Add Secrets to GitHub Repository Settings

1. Navigate to *Settings* within your GitHub repository.
2. Under *Security* on the left-hand side, navigate to *Secrets → Actions*.
3. Click *New Repository Secret*.
4. Give the secret a descriptive name.
    1. Do NOT use dashes or spaces, you can only use camelCase or underscores. 
5. Add the webhook URL as the secret value.
6. Click *Add Secret*.

# Add Secret Variables to GitHub Runner File and Recipes

The GitHub Actions workflow file is autopkg.yml within each repository

1. Under *Configure AutoPkg and Git* create a new line, copying the previous line starting with ``` defaults write com.github.autopkg ```.
2. Create a variable name, which will be used in the actual recipes. In the following example, ``` SANDBOX_API ``` is the variable name (e.g. ``` defaults write com.github.autopkg SANDBOX_API "${{ secrets.SANDBOX_API_AUTOPKG }} ```.
3. Within the curly brackets “{ }” edit the actual name of the secret to be the secret name you created in GitHub settings.
4. Now you can use that variable name in the recipes in this format ``` %SANDBOX_API% ```.

# Use AutoPkg to Download a Package From Another Internal Private Repository

Typically a workflow file can only download packages from public GitHub repositories or a website, but you can use the GitHubReleasesInfoProvider AutoPkg processor to authenticate into a private repository within your GitHub organization.

## Add PAT (Personal Access Token) as a Repository Secret

1. Determine what GitHub account you can use as a service account to connect your AutoPkg repository to the other repositories you would like to access and login to that account
2. Once you are logged into the Github service account, click on *Repositories*
3. Search for the name of the repository that you are pulling the .pkg from 
4. If permissions are setup correctly, you should see an option to click on *Settings* in the repository menu
    1. The GitHub service account you use needs to be added as a collaborator with admin permissions on the repositories in question so you can access settings. If you cannot access settings, ask someone who does to add your PAT as a secret in the GitHub repository for you
5. Once in the repository settings, click on the *Secrets* drop-down in the left-hand menu, and click on *Actions*
6. Click the *New Repository Secret* button
7. Give the new secret a name
8. Paste in your PAT as the value
9. Click *Add Secret*

## Override GITHUB_TOKEN value
1. Navigate to your autopkg.yml workflow file
2. Scroll down to the C*onfigure Autopkg and Git* section
3. Change the value inside the curly brackets to the name you gave your PAT secret in settings
4. Commit your changes

## Credits

[autopkg_tools.py](https://github.com/Gusto/it-cpe-opensource/blob/main/autopkg/autopkg_tools.py) and [autopkg.yml](https://github.com/Gusto/it-cpe-opensource/blob/main/autopkg/workflows/autopkg.yml) from ZenPayroll, Inc., dba Gusto under a BSD 3-clause license.

