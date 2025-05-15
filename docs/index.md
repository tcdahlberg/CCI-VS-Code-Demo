# Using VS Code with Salesforce

## Starting a blank git project directory

- Create an empty folder on you computer
- In VS Code got to File -> Open folder. Navigate to your newly created folder and open it
- Open a terminal (Terminal -> New Terminal)
- In the terminal type git init

## Using Git and Cumulus CI (CCI) to create a new project

Cumulus CI adds the ability to build scratch orgs with dependent code and automation of org settings. [Follow these instructions for installing CCI onto you computer](https://cumulusci.readthedocs.io/en/latest/get-started.html).

- With your folder open type cci project init
- For the most part you can accept the default values presented
- Once the command is complete you will have the stubs of a sfdx project with cci active
- Commit your changes:

    ```PowerShell
    git add .
    git commit -m "Initial Commit"
    ```

- Now you need to connect a dev hub. Dev Hubs are Developer orgs or Production orgs with Dev Hub turned on in them.

    ```PowerShell
    sf org login web --set-default-dev-hub --alias my-hub-org --instance-url https://exciting.sandbox.my.salesforce.com
    ```

    *replace exciting.sandbox.my.salesforce.com with your prod orgs my domain*

- Connect you local git repository to GitHub
  - [Navigate to your git hub account](https://github.com)
  - Click "New" to create a new repository on GitHub
  - Choose a repository owner (this could be you personal account or institution account)
  - Create a repository name unique to the owners account. Don't worry, GitHub will check the uniqueness. Also, repo name do not contain spaces.
  - Choose whether your repo is public or private. Private is visible to only those you wish to see it. Public will be visible to the world, though the world can not adjust or save to it.
  - Click "Create repository"
  - Once the repository is created you will see some instructions on the page on how to use it (hook it up) to your local git repo (or alternately bring the repo down to your local computer git). Since we started on our computer we want to follow the instructions under "…or push an existing repository from the command line." An example:
  
    ```bash
    git remote add origin https://github.com/tcdahlberg/CCI-VS-Code-Demo.git
        
    git branch -M main

    git push -u origin main
    ```

    At this point you may be asked to log in with your browser to GitHub

Now you essentially have two repos running. You have your local git repo that lives on your computer and you have a web based repo that lives on GitHub. When you add and commit locally your GitHub (web based) repo is unaffected. The way you get your local commits to GitHub is to issue the following command in your terminal:

```bash
git push
```

You now have the very basic structure to build a scratch org and start using CCI.

## Spinning up a scratch org

Enter the following command in your terminal to start the scratch org build process.

```PowerShell
cci flow run dev_org --org dev
```

If you have any code in your repository this command would have installed that code. Currently, we have no code so we should have a clean Salesforce org to play with.

To open your freshly built scratch org enter the following command in your terminal:

```PowerShell
cci org browser --org dev
```

Notice that after each command you have to type --org dev . This designates which org structure we are using. CCI builds out some org structures by default: beta, dev, feature, release. For the most part we are concerned with using the dev org structure so we can make life easier for ourselves by defaulting all commands to dev (unless otherwise indicated with a --org). Enter this command in your terminal:

```PowerShell
cci org default dev
```

Now to open the scratch org as we did previously you would only have to enter this command in your terminal:

```PowerShell
cci org browser
```

## Getting changes out of your scratch org

At the end of the scratch org build CCI runs a task called "SnapshotChanges." This builds a snapshot of the current state of the scratch org after everything gets built. The brilliance of this is that any declarative change you make in the org will be out of scope of the snapshot taken. This is how cci will know what changes to capture back into your repository.

To list changes that have been made since your scratch org build in the salesforce org run this task in cci:

```bash
cci task run list_changes
```

Once you have the list you can include or exclude changes you see on that list if you don't want to pull them into your repository. For instance I will never track profiles in my repos so I would add this to my listing task:

```bash
cci task run list_changes --exclude Profile
```

You can also go an inclusive route and use the include to build a list of changes you do want to track.

```bash
cci task run list_changes --exclude Some_Custom_Object__c
```

Once you have a list that looks good you simply have to change the task from list_changes to receive_changes using the excludes and includes you honed while listing changes. The following command will pull all the changes out of your scratch org and save them in your repository:

```bash
cci task run retrieve_changes --exclude Profile
```

## Saving changes to your scratch org

To deploy your whole code base to the scratch org run the following command:

```bash
cci task run deploy --path force-app\main\default
```

