# Deploy Multiple Heroku App from a Single GitHub Repository Using GitHub Action

The problem we are trying to solve here is to (auto) deploy a "mono-repo" to multiple Heroku apps using [GitHub Action](https://github.com/features/actions). 

If you have multiple applications which can be deployed as separate services but all live inside one GitHub repository, you have a monorepo!

It is common to use single git repository for multiple related applications. Such applications reside in subdirectories of the `main` repo.

Notice the structure of codebase in this repository

```text
docs
code
  frontend
  backend
```

The sub-directories `frontend` and `backend` are Gradle Java projects. Both are simple "Hello World" apps written using SparkJava framework. Notice two differences from what you had done in [Module-5](https://cs421sp21.github.io/Backend/index.html):

* There is a `Procfile` in the root folder of each project.
  ```shell
  web: java -jar build/libs/*-0.0.1.jar
  ```
* There is an addition to the `build.gradle` of each project:
  ```groovy
  task stage(dependsOn: ['build', 'clean'])
  build.mustRunAfter clean
  ```

The issue is that Heroku expects one application per repository. This repository is a toy example to use GitHub Actions to build and deploy multiple Heorku apps from a single repository.

To give this a try, download the content and push it to your own repository. You must also make two Heroku apps. You can do this on Heroku website (from your dashboard) or using the Heroku CLI:

```shell
heroku create -a your-app-name-frontend
heroku create -a your-app-name-backend
```

The only other thing you need is to tell GitHub Action to deploy the project inside `code/backend` to the Heroku app you made for it, and the project inside `code/frontend` to the other Heroku app.

You will find in this repository a folder named `.github` that contains a subfolder `workflows`. The latter contains two files: `backend.yml` and `frontend.yml`. Open these files and look at their content. 

The `on` block defines a set of rules telling GitHub when to run each workflow. The code in `backend.yml` for instance says "run this workflow whenever there are pushes to the `main` branch and files have changed in the `code/backend` directory.

The `jobs` section is where we define a workflow to build and deploy the application. Notice the `${{ secrets.HEROKU_API_TOKEN }}`. GitHub allows you to store "secretes" such as API keys, authentication tokens, passwords, etc., in their version of environment variables which is called "secretes"! 

Go to your GitHub repository's "Settings" tab and click on "Secretes". Click "New Secret" to add a new secrete!

You need to add three secretes:

1. `HEROKU_BACKEND_APP_NAME`: the name of your Heroku app which serves the backend
2. `HEROKU_FRONTEND_APP_NAME`: the name of your Heroku app which serves the frontend
3. `HEROKU_API_TOKEN`: this is the authentication token to access Heroku through your account. You can get this token by running `heroku auth:token` in your terminal (assuming Heroku CLI is installed).

Once you have this "secretes" in place, you can make a change in the frontend/backend and push your code to your GitHub repository (to the `main` branch). On GitHub, go to "Actions" tab and watch what happens after you pushed your code! If all goes well, your Gradle projects in the _sub-directories_ will be deployed to their corresponding Heroku apps. Mine are deployed here:
* https://monorepo-multiapp-backend.herokuapp.com/
* https://monorepo-multiapp-frontend.herokuapp.com/