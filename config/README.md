##Contents of This File
* [Overview of Pa11y](#overview-of-pa11y)
* [Pa11y Repo Setup](#pa11y-repo-setup)
* [Setting Up Pa11y on a New Project](#setting-up-pa11y-on-a-new-project)
* [Using the Pa11y Dashboard](#using-the-pa11y-dashboard)
* [Using Pa11y Databases](#using-pa11y-databases)
* [Pa11y Commands](#pa11y-commands)
* [Additional Notes](#additional-notes)

# Overview of Pa11y

[Pa11y](http://pa11y.org) is a suite of tools for testing website accessibility. We're using the [Pa11y Dashboard](https://github.com/pa11y/dashboard) tool for our projects, which uses Node and MongoDB for creating collections of URLs to test against accessibility standards. We've copied the Pa11y Dashboard repo as private repo under the Metal Toad organization, which allows us to have the workflow outlined in the sections below.

Metal Toad's Pa11y repo will hold a sample config file (`config.sample.json`) and the Pa11y README. These are located at `pa11y/config/config.sample.json` and `pa11y/config/README.md`, respectively.

# Pa11y Repo Setup

* Clone the [Metal Toad Pa11y repo](https://github.com/metaltoad/pa11y).
* [Install MongoDB](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/) through Homebrew with `brew update ` and then `brew install mongodb`.
  * When you [create the data directory](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/#run-mongodb), you may have to `sudo chown` for appropriate permissions.
* Navigate into your pa11y repo and run `npm install`

# Setting Up Pa11y on a New Project

**NAMING CONVENTIONS:** Please pay attention to the naming conventions mentioned throughout this README. If you don't, I will know and I will find you, and I will make you pay. Also note that we're using `pa11y`, after the `a11y` convention, *not* `pally`.

* Navigate to the project's `tests` directory. Create a `pa11y` directory with a `config` and a `data` sub-folder. You should end up with `<project root>/tests/pa11y/config` and `<project root>/tests/pa11y/data`.
* Link to the [Pa11y config README](https://github.com/metaltoad/pa11y/blob/master/config/README.md) in the `tests/pa11y` directory.

## Creating a New Config File

* Navigate to `<project>/tests/pa11y/config`. Copy `config.sample.json` from the Pa11y repo to the project repo with `cp ~/Sites/pa11y/config/config.sample.json <project name>.json`.
    * **NAMING CONVENTION:** Name the config file after the project repo name.
* Open your new config file, and change the `siteMessage` value to match your project's name and environment, and the standards you're testing against.
    * **NAMING CONVENTION:** For example, if this config file was for testing AA accessibility standards on a staging site, your `siteMessage` value would be `"<Project> Staging Site: AA Accessibility Check"`.
* In the `webservice` object, replace the `<project name>` placeholder with the actual database name you want to use.
    * **NAMING CONVENTION:** The database name should match the name of the config file. Following our previous examples, the database name would be `mongodb://localhost/<project>`. If you're going to have a Dashboard for different environments on the same project, include the environment in all of your naming. E.g. `<project>-staging.json` for the config file and `mongodb://localhost/<project>-staging` for the database name.
* Create a [symlink](http://apple.stackexchange.com/questions/115646/how-can-i-create-a-symbolic-link-in-terminal) from this config file to the pa11y repo, making sure to use absolute paths.
    * Example: `ln -s ~/Sites/<project>/tests/pa11y/config/<project>.json ~/Sites/pa11y/config`
* Commit the config file to the project repo; no need to commit the symlinked file.

# Using the Pa11y Dashboard

The Pa11y Dashboard is run locally from the pa11y repo. This is where you'll add URLs to test, and run the accessibility tests. You'll also need to be in the pa11y repo to dump and restore the MongoDB databases.

## Dashboard URLs

* In a terminal window, start MongoDB with the `mongod` command
* In a separate terminal window, navigate to the pa11y repo and start the node server with `NODE_ENV=<project> node index.js`
    * The parameter for `NODE_ENV` is the name of the config file. So if we wanted the dashboard for our `<project>.json` config file, you'd run`NODE_ENV=<project> node index.js`.
* Go to `http://localhost:4000/` in your browser.
* Click 'Add new URL'.
* Fill out the appropriate fields, and then click 'Add URL'. **NOTE:** After you add and save a new URL, the URL and Standard fields *cannot* be edited later.
    * Name: Be brief but clear
    * URL: The URL you want to test
    * Standard: Select the standard that you want to test against.
    * Timeout: Use a high enough value that it won't time out.
    * Wait: If you want no wait time, you have to enter a value of 0
    * Username & Password: Even sites with basic auth don't require these to be filled out. Leave blank unless it's preventing you from running the tests.
    * HTTP Headers, Hide Elements: Fill out as desired
* You can run Pa11y two ways: Click on a URL and then click 'Run Pa11y' on the next page; or hover over the URL, click the gear icon, and then select 'Run Pa11y'.

# Using Pa11y Databases

By design, Pa11y accessibility tests are run locally. We use some MongoDB commands so we can commit the database information, which means the same Dashboards can be viewed by all of our developers.

## Dumping the Database

Once you've set up the project's dashboard by adding URLs and running Pa11y against each of them, you'll need to dump the database into the project repo.
* From the pa11y repo, run `mongodump --db=<project> --out=/Users/<user>/Sites/<project>/tests/pa11y/data`.
    * **NOTE:** You do have to use the full `/Users/<user>` instead of `~` for the `out` parameter.
    * The `db` parameter comes from the database name in our config file. Using the previous Metal Toad repo example, we named the database `mongodb://localhost/mtm-d8-rebuild`, so the full command would be `mongodump --db=<project> --out=/Users/<user>/Sites/<project>/tests/pa11y/data`
* In the project's repo, verify that the database folder was created. You should see `<project>/tests/pa11y/data/<project db>`.
    * The `<project db>` folder should have 4 files: `results.bson`, `results.metadata.json`, `tasks.bson`, and `tasks.metadata.json`. The tasks refer to the URLs you added in the Dashboard, and the results refer to the results of running Pa11y against those URLs.
* Commit these changes to the repo and put in a PR, so the next user can pull down the database. **NAMING CONVENTION:** Let's use `pa11y` for the branch name, in case someone will be using the Dashboard before the PR is merged. Don't make the next person guess which branch they need to checkout.

## Restoring the Database

Restoring the database is what allows a different user to access the project's Pa11y Dashboard. The steps here assume that the person pulling the Pa11y database already has the Pa11y repo set up.

* Navigate to the *project repo* and pull the latest code, so you have the config file. If the PR was merged by the time you've gotten to this step, you'll have the database dump on the dev branch. Otherwise, hope that Past You or your coworkers used a logical branch name for their commit.
  * If this is your first time using Pa11y on a particular project, you'll need to create a local [symlink](http://apple.stackexchange.com/questions/115646/how-can-i-create-a-symbolic-link-in-terminal) from the config file to your local pa11y repo, making sure to use absolute paths. No need to commit that symlinked file.
  * Example: `ln -s ~/Sites/mtm-d8-rebuild/tests/pa11y/config/mtm-d8-rebuild.json ~/Sites/pa11y/config`
* Navigate to the pa11y repo and pull the latest code.
* Get the database dump by running `mongorestore --db=<project> /Users/<user>/Sites/<project>/tests/pa11y/data/<project db>`.
    * For example, the full command would be `mongorestore --db=<project> /Users/<user>/Sites/<project>/tests/pa11y/data/<project>`

Now you're ready to access the project's Pa11y Dashboard! Go back up to the [Using the Pa11y Dashboard](#using-the-pa11y-dashboard) section for steps. If you make changes to the existing URLs, add or delete URLs, or run Pa11y on any of the URLs, you'll need to do another database dump.

# Pa11y Commands

* `NODE_ENV=<project> node index.js`: starts the Pa11y Dashboard server
* `mongod`: starts mongodb
* `mongodump --db=<project> --out=/Users/<user>/Sites/<project>/tests/pa11y/data`: database dump
* `mongorestore --db=<project> /Users/<user>/Sites/<project>/tests/pa11y/data/<project>`: database restore

