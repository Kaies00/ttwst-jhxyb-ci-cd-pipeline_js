# coding-project-template
::page{title="Lab (Option B: JavaScript): Using GitHub Actions - Part 1"}

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-CD0215EN-SkillsNetwork/images/IDSN-logo.png" width="200">

**Estimated time needed:** 30 minutes

Welcome to the hands-on lab for **Using GitHub Actions -  Setting up Workflow**. In this part, you will build a workflow in a GitHub repository using GitHub Actions. You will create an empty workflow file in Step 1 and add events and a job runner in the following steps. You will subsequently finish the workflow in the next lab called **Using GitHub Actions - Part 2**. Ensure you finish this lab before starting part 2.

## Learning objectives
After completing this lab, you will be able to:

- Create a GitHub workflow to run your CI pipeline
- Add events to trigger the workflow
- Add a job to the workflow
- Add a job runner to the job
- Add a container to the job runner

## Prerequisites
You will need the following to complete the exercises in this lab:
- A basic understanding of YAML
- A GitHub account
- An intermediate-level knowledge of CLIs

---

::page{title="Generate GitHub personal access token"}

You have a little preparation to do before you can start the lab.

## Generate a personal access token
You will fork and clone a repo in this lab using the `gh` CLI tool. You will also push changes to your cloned repo at the end of this lab. This requires you to authenticate with GitHub using a `personal access token`. Follow the steps here to generate this token and save it for later use:

1. Navigate to [GitHub Settings](https://github.com/settings/tokens "GitHub Settings") of your account.
2. Click **Generate new token (classic)** to create a personal access token.
![](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/0epcRy73ZeZdB-nm7kOccg/Screenshot%202025-01-21%20at%201-02-07%E2%80%AFAM.png)
3. Give your token a descriptive name and optionally change the expiration date.
![](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-CD0215EN-SkillsNetwork/images/actions_token_name_expiry.png "Set token name and expiration date")
4. Select the minimum required scopes needed for this lab: repo, read:org, and workflow.
![](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-CD0215EN-SkillsNetwork/images/actions_token_permissions.png "Select the scopes")
5. Click **Generate token**.
![](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-CD0215EN-SkillsNetwork/images/actions_token_generate_finish.png "Generate token")
6. Make sure you copy the token and paste it somewhere safe as you will need it in the next step. **WARNING: You will not be able to see it again.**
![](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-CD0215EN-SkillsNetwork/images/actions_token_generate_warning.png "Warning message")
`Warning: Keep your tokens safe and protect them like passwords.`

If you lose this token at any time, repeat the above steps to regenerate the token.

::page{title="Fork and clone the repository"}

## Open a terminal

Open a terminal window by using the menu in the editor: Terminal > New Terminal.

![](http://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-CD0241EN-SkillsNetwork/labs/module2/images/01_terminal.png "New Terminal option on Terminal menu")

In the terminal, if you are not already in the /home/project folder, change to your project folder now.
```shell
cd /home/project
```


## Authenticate with GitHub

First, let\'s run the following commands to install GitHub CLI.

```shell
sudo apt update
sudo apt install gh
```

Then, run the following command to authenticate with GitHub in the terminal. You will need the `GitHub Personal Token` you created in the previous step.

```shell
gh auth login

```

You will be taken through a guided experience as shown here:
> What account do you want to log into? GitHub.com
> What is your preferred protocol for Git operations? HTTPS
> Authenticate Git with your GitHub credentials. Yes
> How would you like to authenticate GitHub CLI? Paste an authentication token.
> Paste your authentication token: ****************************************
> You will be logged into GitHub as your account user.

After you have authenticated successfully, you will need to fork and clone [this GitHub](https://github.com/ibm-developer-skills-network/ttwst-jhxyb-ci-cd-pipeline_js "GitHub") repo in the terminal. You will then create a workflow to trigger GitHub Actions in your forked version of the repository.

## Fork and clone the reference Repo
```shell
gh repo fork ibm-developer-skills-network/ttwst-jhxyb-ci-cd-pipeline_js
```
`Note` Once you run the command, it will prompt you to clone the fork. Type Yes to proceed.

Your output should look similar to the image below:
![Screenshot 2025-05-25 at 11.20.14 PM.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/l74aygDk1Kl595pNwVXEtQ/Screenshot%202025-05-25%20at%2011-20-14%E2%80%AFPM.png)
> ###### **Important:** Pull request
 When making a pull request, make sure that your request is merging with your fork because the pull request of a fork will default to come back to [this](https://github.com/ibm-developer-skills-network/ttwst-jhxyb-ci-cd-pipeline_js "this") repo,  not your fork.

## Change to the lab folder
Once you have cloned the repository, change to the directory named `ttwst-jhxyb-ci-cd-pipeline_js`

```shell
cd ttwst-jhxyb-ci-cd-pipeline_js
```

List the contents of this directory to see the artifacts for this lab.

```shell
ls -l
```

The directory should look like the listing below:

![Screenshot 2025-05-25 at 11.21.43 PM.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/ZUGxfDp4rh6isl2GZ_PVEg/Screenshot%202025-05-25%20at%2011-21-43%E2%80%AFPM.png)

You can also view the files cloned in the file explorer.
![Screenshot 2025-05-25 at 11.23.41 PM.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/koYChOfFkY_JSl2HveGTMw/Screenshot%202025-05-25%20at%2011-23-41%E2%80%AFPM.png)

You are now ready to start the lab.

### Optional

If working in the terminal becomes difficult because the command prompt is very long, you can shorten the prompt using the following command:

```bash
export PS1="[\[\033[01;32m\]\u\[\033[00m\]: \[\033[01;34m\]\W\[\033[00m\]]\$ "
```

---

::page{title="Step 1: Create a workflow"}

To get started, you need to create a workflow yaml file. The first line in this file will define the name of the workflow that shows up in GitHub Actions page of your repository.

## Your task
1. Open the terminal and ensure you are in the `ttwst-jhxyb-ci-cd-pipeline_js` directory.
	<details>
		<summary>Click here for a hint.</summary>

	```shell
	cd /home/project/ttwst-jhxyb-ci-cd-pipeline_js
	```

	</details>
1. Create the directory structure `.github/workflows` and create a file called `workflow.yml`.
	<details>
			<summary>Click here for a hint.</summary>

	```shell
	mkdir -p .github/workflows
	touch .github/workflows/workflow.yml
	```

	</details>
1. Every workflow starts with a name. The name will be displayed on the Actions page and on any badges. Give your workflow the name `CI workflow` by adding a `name:` tag as the first line in the file.
	<details>
			<summary>Click here for a hint.</summary>

	```yaml
	name: {insert name here}
	```

	</details>

::openFile{path="/home/project/ttwst-jhxyb-ci-cd-pipeline_js/.github/workflows/workflow.yml"}


Double-check that your work matches the solution below.

### Solution

<details>
	<summary>Click here for the answer.</summary>
Replace the workflow.yml file with the code snippet below. You can also copy relevant parts of the code. Be sure to indent properly:

```yaml
name: CI workflow
```

</details>

---

::page{title="Step 2: Add event triggers"}

Event triggers define which events can cause the workflow to run. You will use the `on:` tag to add the following events:
- Run the workflow on every push to the main branch
- Run the workflow whenever a pull request is created to the main branch.

## Your Task

1. Add the `on:` keyword to the workflow at the same level of indentation as the `name:`.
	<details>
		<summary>Click here for a hint.</summary>

	```yaml
	on:
	```

	</details>
3. Add `push:` event as the first event that can trigger the workflow. This is added as the child element of `on:` so it must be indented under it.
	<details>
		<summary>Click here for a hint.</summary>

	```yaml
	on:
	  {insert first event name here}:
	```

	</details>
4. Add the `"main"` branch to the push event. You want the workflow to start every time somebody pushes to the main branch. This also includes merge events. You do this by using the `branches:` keyword followed by a list of branches either as `[]` or `-`
	<details>
		<summary>Click here for a hint.</summary>

	```yaml
	on:
	  push:
	    branches: [ {insert branch name here} ]
	```

	</details>
5. Add a `pull_request:` event similar to the push event you just finished. It should be triggered whenever the user makes a pull request on the main branch.
	<details>
		<summary>Click here for a hint.</summary>

	```yaml
	on:
	  push:
	    branches: [ "main" ]
	  {insert second event name here}:
	    branches: [ {insert branch name here} ]
	```
	</details>
Double-check that your work matches the solution below.

### Solution

<details>
	<summary>Click here for the answer.</summary>
Replace the workflow.yml file with the code snippet below. You can also copy relevant parts of the code. Be sure to indent properly:

```yaml
name: CI workflow

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
```
</details>

---

::page{title="Step 3: Add a job"}

You will now add a job called `build` to the workflow file. This job will run on the `ubuntu-latest` runner. Remember, a job is a collection of steps that are run on the events you added in the previous step.

## Your task
1. First you need a job. Add the `jobs:` section to the workflow at the same level of indentation as the `name` (i.e., no indent).
	<details>
		<summary>Click here for a hint.</summary>

	```yaml
	jobs:
	```

	</details>
1. Next, you need to name the job. Name your job `build:` by adding a new line under the `jobs:` section.
	<details>
		<summary>Click here for a hint.</summary>

	```yaml
	jobs:
  	  {insert job name here}:
	```

	</details>
1. Finally, you need a runner. Tell GitHub Actions to use the `ubuntu-latest` runner for this job. You can do this by using the `runs-on:` keyword.
	<details>
		<summary>Click here for a hint.</summary>

	```yaml
	jobs:
	  build:
	    runs-on: {insert runner name here}
	```

	</details>

Double-check that your work matches the solution below.

### Solution

<details>
	<summary>Click here for the answer.</summary>
Replace the workflow.yml file with the code snippet below. You can also copy relevant parts of the code. Be sure to indent properly:

```yaml
name: CI workflow

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
```
</details>

---

::page{title="Step 4: Target Node.js 20"}

It is important to consistently use the same version of dependencies and operating systems for all phases of development, including the CI pipeline. This project was developed on Node.js 20, so you need to ensure that the CI pipeline also runs on the same version of Node.js. You will accomplish this by running your workflow in a container inside the GitHub action.

## Your task

1. Add a `container:` section under the `runs-on:` section of the build job, and tell GitHub Actions to use `node:20-alpine` as the image.

### Hint

<details>
	<summary>Click here for a hint.</summary>

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    container: {insert container name here}
```

</details>

Double-check that your work matches the solution below.

### Solution

<details>
	<summary>Click here for the answer.</summary>
Replace the workflow.yml file with the code snippet below. You can also copy relevant parts of the code. Be sure to indent properly:

```yaml
name: CI workflow

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    container: node:20-alpine
```
</details>

---

::page{title="Step 5: Save your work"}

It is now time to save your work and return it to your forked GitHub repository.

## Your task

1. Configure the Git account with your email and name using the `git config --global user.email` and `git config --global user.name` commands.

	<details>
		<summary>Click here for a hint.</summary>

	Open the terminal and configure your email:
	```
	git config --global user.email "you@example.com"
	```

	Open the terminal and configure your user name
	```
	git config --global user.name "Your Name"
	```

	</details>

1. The next step is to stage all the changes you made in the previous exercises and push them to your forked repo on GitHub.

	<details>
		<summary>Click here for a hint.</summary>

	You can use the following commands to commit your changes to staging and then push to your forked repository:
	```shell
	git add -A
	git commit -m "COMMIT MESSAGE"
	git push
	```
	</details>

Your output should look similar to the image below:

### Solution

![Screenshot 2025-05-25 at 11.39.17 PM.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/34ZW8Ed1GKu_t-xSY7fv4A/Screenshot%202025-05-25%20at%2011-39-17%E2%80%AFPM.png)

You are done with part 1 of the lab. However, if you look at the `Actions` tab in your forked repository, you will notice the GitHub action was triggered and has failed. The action was triggered because you `pushed` the code to the `main` branch of the repository. It failed as you have not finished the workflow yet. You will add the remaining steps in part 2 of the lab so the workflow runs successfully. You can ignore this error at this time.

![Screenshot 2025-05-25 at 11.40.34 PM.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/aKP90vDIqn62PHisHDzV_g/Screenshot%202025-05-25%20at%2011-40-34%E2%80%AFPM.png)

---

::page{title="Conclusion"}

Congratulations! In this lab, you started building your Continuous Integration pipeline. This pipeline will run automatically when you commit your code to the GitHub repository based on the events described in the workflow.

You successfully created a GitHub Actions workflow and added an empty job. You can now proceed to extend the CI pipeline by adding steps to build dependencies, test your code, and report test coverage.

## Author(s)
Harsh

<!-- ## Change Log
| Date | Version | Changed by | Change Description |
|------|--------|--------|---------|
| 2025-05-25 | 0.1 | Harsh| Initial version created |
| 2025-05-30 | 0.2 | Pornima More | QC pass with edits |
| 2025-07-22 | 0.3     | Sapthashree | Updated the Title as per Rav's feedback |-->

## <h3 align="center"> © IBM Corporation. All rights reserved. <h3/>
#-----------------------------------------
::page{title="Lab (Option B: JavaScript): Using GitHub Actions - Part 2"}

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-CD0215EN-SkillsNetwork/images/IDSN-logo.png" width="200">

**Estimated time needed:** 30 minutes

In this lab, you will build the Continuous Integration pipeline by setting up the steps under job in your workflow. Please ensure you complete setting up the workflow, before starting this lab.

You will add the following steps to the job in this lab:
- Check out the code
- Install dependencies
- Lint code with ESLint
- Run unit tests with Jest

## Learning objectives
After completing this lab, you will be able to:

- Explain components of a job in a GitHub Actions workflow
- Use predefined GitHub actions
- Run multiline inline code in a step
- Explore action logs in the GitHub UI

---

::page{title="Prerequisites"}

- It is highly recommended that you complete the lab titled \'\'Using GitHub Actions - Setting up workflow.\'\' If you have not finished the lab, please finish that before starting this lab.
- A basic understanding of YAML.
- A GitHub account.
- An intermediate-level knowledge of CLIs.


## Pre-work

1. Generate a personal access token on [GitHub Settings](https://github.com/settings/tokens "GitHub Settings"), if you already don\'t have one.
2. Open a terminal window by using the menu in the editor: Terminal > New Terminal.

	First, let\'s run the following commands to install GitHub CLI.

```shell
sudo apt update
sudo apt install gh
```

3. Run the following command to authenticate with GitHub in the terminal with the personal access token.

```shell
gh auth login
```

You will be taken through a guided experience, as shown here:
> What account do you want to log into? **GitHub.com**
> What is your preferred protocol for Git operations? **HTTPS**
> Authenticate Git with your GitHub credentials. **Yes**
> How would you like to authenticate GitHub CLI? **Paste an authentication token.**
> Paste your authentication token: ****************************************
> You will be logged into GitHub as your account user.

4. Clone your copy of the forked [GitHub repository](https://github.com/ibm-developer-skills-network/ttwst-jhxyb-ci-cd-pipeline_js "GitHub"). This creates the `ttwst-jhxyb-ci-cd-pipeline_js` subdirectory under `/home/project` directory.

>If you already have cloned this repository under /home/project, you can skip this step.


5. If you don\'t have `.github/workflows/workflow.yml` in `ttwst-jhxyb-ci-cd-pipeline_js` directory, create `workflow.yml` file in the `/home/project/ttwst-jhxyb-ci-cd-pipeline_js/.github/workflows` directory with the following code:

```yaml
name: CI workflow

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    container: node:20-alpine
```



::page{title="Step 1: Add the checkout step"}

You will now add your first step to the job. The first task is to check out the code from the repository. You will use the predefined `actions/checkout@v3` action to do this. You can learn more about this action on the [GitHub page](https://github.com/actions/checkout "GitHub page").

## Your task
1. Add the `steps:` section at the end of the `build` job in the workflow file.
	<details>
		<summary>Click here for a hint.</summary>

	```yaml
	steps:
	```

	</details>
1. Add the checkout step as the first step. Give it the `name: Check out` and call the action  `actions/checkout@v3` by using the `uses:` keyword.
	<details>
		<summary>Click here for a hint.</summary>

	```yaml
	steps:
	  - name: {insert step name here}
	    uses: {insert action name here}
	```

	</details>

Double-check that your work matches the solution below.

### Solution

<details>
	<summary>Click here for the answer.</summary>
Replace the workflow.yml file with the code snippet below. You can also copy relevant parts of the code. Be sure to indent properly:

```yaml
name: CI workflow

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    container: node:20-alpine
    steps:
	  - name: Checkout
        uses: actions/checkout@v3
```
</details>

---

::page{title="Step 2: Install dependencies"}

Now that you have checked out the code, the next step is to install the dependencies. This application uses `npm`, the Node.js package manager, to install all the dependencies from the `package.json` file. The `node:20-alpine` container you are using already has the npm package manager installed.

The commands that you will use in this step are:

```
npm ci
```


This command installs all the dependencies listed in the package.json and package-lock.json files. The `ci` command, which stands for \"clean install,\" is specifically designed for CI/CD pipelines. Unlike `npm install`, it does not modify the package-lock.json file and installs exactly what's specified there


## Your task

1. Add a new named step after the `checkout` step. Name this step `Install dependencies`.
	<details>
		<summary>Click here for a hint.</summary>

	```yaml
	  - name: {insert step name here}
	```

	</details>
1. Next, you want to run the command to install the dependencies. You can run this using the `run:` keyword

	<details>
		<summary>Click here for a hint.</summary>

	```yaml
	  - name: Install dependencies
	    run: |
          {insert code here}
	```

	</details>

Double-check that your work matches the solution below.

### Solution

<details>
	<summary>Click here for the answer.</summary>
Replace the workflow.yml file with the code snippet below. You can also copy relevant parts of the code.

```yaml
name: CI workflow

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    container: node:20-alpine
    steps:
	  - name: Checkout
        uses: actions/checkout@v3

	  - name: Install dependencies
        run: |
          npm ci
```
</details>


::page{title="Step 3: Lint with ESLint"}

It is always a good idea to add quality checks to your CI pipeline. This is especially true if you are working on an open source project with many different contributors. This makes sure that everyone who contributes is following the same style guidelines.

The next step is to use `ESLint` to lint the source code. Linting is checking your code for syntactical and stylistic issues. Some examples are spacing, using spaces or tabs for indentation, locating uninitialized or undefined variables, and missing brackets. The `eslint` library should be installed as a development dependency in the package.json file.

The ESLint command will analyze your JavaScript code according to the rules defined in your .eslintrc.* configuration file.

```
npm run lint
```

This command runs the lint script defined in your package.json file, which typically executes eslint on your source code.




## Your task

1. Add a new named step after the `Install dependencies` step. Call this step `Lint with ESLint`.
	<details>
		<summary>Click here for a hint.</summary>

	```yaml
	steps:
	  - name: {insert step name here}

	```
	</details>
1. Next, you want to run the ESLint command to lint your code.
	```
	npm run lint
	```
	You can run inline commands using the `run` keyword with the pipe `|` operator.

	<details>
		<summary>Click here for a hint.</summary>

	```yaml
	steps:
	  - name: Lint with ESLint
	    run: |
          {insert first command here}
	```
	</details>

Double-check that your work matches the solution below.

### Solution

<details>
	<summary>Click here for the answer.</summary>
Replace the workflow.yml file with the code snippet below. You can also copy relevant parts of the code. Be sure to indent properly:

```yaml
name: CI workflow

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    container: node:20-alpine
    steps:
	  - name: Checkout
        uses: actions/checkout@v3

	  - name: Install dependencies
        run: |
          npm ci

	  - name: Lint with ESLint
        run: |
          npm run lint

```
</details>

---

::page{title="Step 4: Test code coverage with Jest"}

You will use `Jest` in this step to unit test the source code. Jest is a delightful JavaScript testing framework with a focus on simplicity. It works with projects using Babel, TypeScript, Node, React, Angular, Vue, and more.

Jest is configured via the jest.config.js file or in package.json to automatically collect code coverage information. At the end of the test run, you should see a percentage of coverage report.

## Your task

1. Add a new named step after the `Lint with ESLint` step. Call this step `Run unit tests with Jest`.
	<details>
		<summary>Click here for a hint.</summary>

	```yaml
	steps:
	  - name: {insert step name here}

	```

	</details>
1. Next, you want to run the Jest command to test your code and report on code coverage.

	```
	npm test -- --coverage
	```
	This command runs Jest with the coverage flag enabled, which generates a code coverage report.

	<details>
		<summary>Click here for a hint.</summary>

	```yaml
	  - name: Run unit tests with Jest
	    run: {insert command here}
	```

	</details>

Double-check that your work matches the solution below.

### Solution

<details>
	<summary>Click here for the answer.</summary>
Replace the workflow.yml file with the code snippet below. You can also copy relevant parts of the code. Be sure to indent properly:

```yaml
name: CI workflow

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    container: node:20-alpine
    steps:
	  - name: Checkout
        uses: actions/checkout@v3

	  - name: Install dependencies
        run: |
          npm ci

	  - name: Lint with ESLint
        run: |
          npm run lint

      - name: Run unit tests with Jest
        run: npm test -- --coverage

```
</details>

---

::page{title="Step 5: Push code to GitHub"}

To test the workflow and the CI pipeline, you need to commit the changes and push your branch back to the GitHub repository. As described earlier, each new push to the main branch should trigger the workflow.

## Your task

1. Configure the Git account with your email and name using the `git config --global user.email` and `git config --global user.name` commands.

	<details>
		<summary>Click here for a hint.</summary>

	Open the terminal and configure your email:
	```
	git config --global user.email "you@example.com"
	```

	Open the terminal and configure your user name
	```
	git config --global user.name "Your Name"
	```

	</details>

1. The next step is to stage all the changes you made in the previous exercises and push them to your forked repo on GitHub.

	<details>
		<summary>Click here for a hint.</summary>

	You can use the following commands to commit your changes to staging and then push to your forked repository:
	```shell
	git add -A
	git commit -m "COMMIT MESSAGE"
	git push
	```
	</details>

Your output should look similar to the image below:

### Solution

![Screenshot 2025-05-26 at 1.21.30 AM.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/_wWonMTjwn5C6g14NUKobg/Screenshot%202025-05-26%20at%201-21-30%E2%80%AFAM.png)

---

::page{title="Step 6: Run the workflow"}

You can run the workflow now. Open the **Actions** tab in the forked repository in your GitHub account. Click the ***CI workflow*** action. Pushing changes in the previous step should have triggered a build action and it should be visible in the Actions tab.

![Screenshot 2025-05-26 at 1.43.37 AM.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/xncSRifVSkicdUL6yWQYKA/Screenshot%202025-05-26%20at%201-43-37%E2%80%AFAM.png)

Click the most recent `workflow run` to open the details page. You should see your workflow completed successfully:

![Screenshot 2025-05-26 at 1.44.59 AM.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/7sk17xhoMoh8rlalOIjXww/Screenshot%202025-05-26%20at%201-44-59%E2%80%AFAM.png)

You can now click the `build` job to see the logs for each step:

![Screenshot 2025-05-26 at 1.46.26 AM.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/r0rW94MZBnKE-WvQF94SpQ/Screenshot%202025-05-26%20at%201-46-26%E2%80%AFAM.png)

Spend some time here to expand each section:

![Screenshot 2025-05-26 at 1.48.04 AM.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/r0VOf4sCSCTRdCsAsh7Zew/Screenshot%202025-05-26%20at%201-48-04%E2%80%AFAM.png)

If the workflow completes, you will see the `Complete job` step at the end of the build job logs.

![Screenshot 2025-05-26 at 1.48.59 AM.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cMsqHwWWjAykcEGpUSWe9Q/Screenshot%202025-05-26%20at%201-48-59%E2%80%AFAM.png)


::page{title="Conclusion"}

Congratulations! You are now able to use GitHub Actions to create workflows for the CI pipeline.

In this lab, you added the steps to your Continuous Integration pipeline for your application. As a result, your application is automatically built and tested when you commit your code to the GitHub repository.

## Next steps

Try to set up a GitHub action for your own project that checks out the code, runs unit tests, and performs linting on every push request to the default branch.

## Author(s)
Tapas Mandal

### Other Contributor(s)
Captain Fedora
John Rofrano

<!---
## Change Log
| Date | Version | Changed by | Change Description |
|------|--------|--------|---------|
| 2025-07-22 | 0.1     | Sapthashree | Updated the Title as per Rav's feedback |
-->
## <h3 align="center"> © IBM Corporation. All rights reserved. <h3/>


