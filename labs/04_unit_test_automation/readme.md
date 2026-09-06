::page{title="Lab (Option B: JavaScript): Integrating Unit Test Automation"}

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-CD0215EN-SkillsNetwork/images/IDSN-logo.png" width="200">

**Estimated time needed:** 30 minutes

Welcome to the hands-on lab for **Integrating Unit Test Automation**. In this lab, you will take the cloned code from the previous pipeline step and run linting and unit tests against it to ensure it is ready to be built and deployed.Many commonly used tasks are available in the Tekton Catalog, which is hosted on GitHub and indexed on [Artifact Hub](https://artifacthub.io/packages/search?repo=tekton-catalog-tasks "Artifact Hub") for discovery.

## Learning objectives

After completing this lab, you will be able to:

- Create a custom ESLint task to install the eslint task
- Describe the parameters required to use the eslint task
- Use the eslint task in a Tekton pipeline to lint your JavaScript code
- Create a test task from scratch and use it in your pipeline

::page{title="Set up the lab environment"}

You have a little preparation to do before you can start the lab.

## Open a terminal

Open a terminal window by using the menu in the editor: Terminal > New Terminal.

![Terminal](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-CD0215EN-SkillsNetwork/images/01_terminal.png "New Terminal option on Terminal menu")

In the terminal, if you are not already in the `/home/project` folder, change to your project folder now.

```bash
cd /home/project
```

## Clone the code repo

Now, get the code that you need to test. To do this, use the `git clone` command to clone the Git repository:

```bash
git clone https://github.com/ibm-developer-skills-network/ttwst-jhxyb-ci-cd-pipeline_js.git
```

Your output should look similar to the image below:

![Screenshot 2025-05-27 at 3.01.58 AM.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/sIML2tqr97EbpbfG0rPp-g/Screenshot%202025-05-27%20at%203-01-58%E2%80%AFAM.png)

## Change to the labs directory

Once you have cloned the repository, change to the labs directory.

```bash
cd ttwst-jhxyb-ci-cd-pipeline_js/labs/04_unit_test_automation/
```
## Navigate to the labs folder

Navigate to the `labs/04_unit_test_automation` folder in the left explorer panel. All of your work will be with the files in this folder.

![Screenshot 2025-05-27 at 3.05.38 AM.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/EmVeF9-sPfwFooOwqPLB-w/Screenshot%202025-05-27%20at%203-05-38%E2%80%AFAM.png)

You are now ready to continue installing the **Prerequisites**.

### Optional

If working in the terminal becomes difficult because the command prompt is very long, you can shorten the prompt using the following command:

```bash
export PS1="[\[\033[01;32m\]\u\[\033[00m\]: \[\033[01;34m\]\W\[\033[00m\]]\$ "
```

---

::page{title="Prerequisites"}

This lab requires the installation of the tasks introduced in previous labs. To be sure, apply the previous tasks to your cluster before proceeding. Reissuing these commands will not hurt anything:

### Establish the tasks

```bash
kubectl apply -f tasks.yaml
```
The `git-clone` task is part of the **Tekton Catalog** and is indexed on **Artifact Hub** for discovery. You can install the task using either of the following options.

**Option 1: Discover via Artifact Hub and apply the raw manifest**

```bash
kubectl apply -f https://github.com/tektoncd/catalog/raw/main/task/git-clone/0.10/git-clone.yaml
```

**Option 2: Install directly from the Tekton Catalog**

Artifact Hub indexes tasks from the official Tekton Catalog hosted on GitHub. You can install the same task directly from the source repository:

```bash
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-clone/0.9/git-clone.yaml
```

Both options install the git-clone task into your cluster under the currently active namespace.

Check that you have all of the previous tasks installed:

```bash
tkn task ls
```

You should see the output similar to this:
```
NAME        DESCRIPTION              AGE
checkout                             2 minutes ago
echo                                 2 minutes ago
git-clone   These Tasks are Git...   2 minutes ago
```

### Establish the workspace

You also need a PersistentVolumeClaim (PVC) to use as a workspace. Apply the following `pvc.yaml` file to establish the PVC:

```bash
kubectl apply -f pvc.yaml
```

You should see the following output:
> Note: If the PVC already exists, the output will say **unchanged** instead of **created**. This is fine.

```text
persistentvolumeclaim/pipelinerun-pvc created
```

You can now reference this persistent volume claim by its name `pipelinerun-pvc` when creating workspaces for your Tekton tasks.

You are now ready to continue with this lab.

---

::page{title="Step 0: Check for cleanup"}

Please check as part of Step 0 for the new `cleanup` task that has been added to `tasks.yaml` file.

When a task causes a compilation of the JavaScript code, it leaves behind build files and node_modules that are owned by the specific user. For consecutive pipeline runs, the git-clone task tries to empty the directory but needs privileges to remove these files, and this `cleanup` task takes care of that.

The `init` task is added to the `pipeline.yaml` file, which runs every time before the `clone` task.

Check the `tasks.yaml` file, which has the new `cleanup` task updated.

### Check the updated cleanup task

<details>
	<summary>Click here.</summary>

```---
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: cleanup
spec:
  description: This task will clean up a workspace by deleting all of the files.
  workspaces:
    - name: source
  steps:
    - name: remove
      image: alpine:3
      env:
        - name: WORKSPACE_SOURCE_PATH
          value: $(workspaces.source.path)
      workingDir: $(workspaces.source.path)
      securityContext:
        runAsNonRoot: false
        runAsUser: 0
      script: |
        #!/usr/bin/env sh
        set -eu
        echo "Removing all files from ${WORKSPACE_SOURCE_PATH} ..."
        # Delete any existing contents of the directory if it exists.
        #
        # We don't just "rm -rf ${WORKSPACE_SOURCE_PATH}" because ${WORKSPACE_SOURCE_PATH} might be "/"
        # or the root of a mounted volume.
        if [ -d "${WORKSPACE_SOURCE_PATH}" ] ; then
          # Delete non-hidden files and directories
          rm -rf "${WORKSPACE_SOURCE_PATH:?}"/*
          # Delete files and directories starting with . but excluding ..
          rm -rf "${WORKSPACE_SOURCE_PATH}"/.[!.]*
          # Delete files and directories starting with .. plus any other character
          rm -rf "${WORKSPACE_SOURCE_PATH}"/..?*
        fi
```
</details>

Check the `pipeline.yaml` file, which is updated with `init` that uses the `cleanup` task.

### Check the updated init task

<details>
	<summary>Click here.</summary>

```---
tasks:
    - name: init
      workspaces:
        - name: source
          workspace: pipeline-workspace
      taskRef:
        name: cleanup
```
</details>

::page{title="Step 1: Add the ESLint task"}

Your pipeline has a placeholder for a `lint` step that uses the `echo` task. Now, it is time to replace it with a real linter.

You are going to use `ESLint` to lint your JavaScript code. Since there isn\'t a pre-built ESLint task in Tekton Catalog, you will create your own ESLint task.

First, let\'s add a custom ESLint task to your tasks.yaml file:

```bash
# We'll add this task manually since it's not available in Tekton Catalog
```



::page{title="Step 2: Create ESLint task"}

Now, you will add a custom ESLint task to the `tasks.yaml` file.

Open the tasks.yaml file and add the following ESLint task:

::openFile{path="/home/project/ttwst-jhxyb-ci-cd-pipeline_js/labs/04_unit_test_automation/tasks.yaml"}

### Your task

1. Add a new task called eslint to the tasks.yaml file. Remember, each new task must be separated using three dashes---on a separate line.
1. The task should:
   - Have a workspace named source
   - Accept parameters for image (default: node:18-alpine) and args
   - Run ESLint with the specified arguments



### Solution

<details>
	<summary>Click here for the answer.</summary>

```yaml
---
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: eslint
spec:
  workspaces:
    - name: source
  steps:
    - name: install-dependencies
      image: node:18
      workingDir: $(workspaces.source.path)
      script: |
        npm install
    - name: run-eslint
      image: node:18
      workingDir: $(workspaces.source.path)
      script: |
        npx eslint .
```
</details>

Apply these changes to your cluster:

```bash
kubectl apply -f tasks.yaml
```


---

::page{title="Step 3: Modify the pipeline to use ESLint"}

Now, you will modify the `pipeline.yaml` file to use the new eslint task.

Open the pipeline.yaml file and modify the lint task:

Edit the `pipeline.yaml` file:

::openFile{path="/home/project/ttwst-jhxyb-ci-cd-pipeline_js/labs/04_unit_test_automation/pipeline.yaml"}

### Your task

1. Add the `workspaces:` keyword to the lint task after the task `name:` but before the `taskRef:`
1. Specify the workspace `name:` as `source`
1. Specify the `workspace:` reference as `pipeline-workspace`
1. Change the `taskRef:` from echo to reference the eslint task
1. Change the parameters to use image and args instead of message



### Hint

<details>
	<summary>Click here for a hint.</summary>

```yaml
	- name: lint
      workspaces:
        - name: {workspace `name:`}
          workspace: {`workspace:` reference }
      taskRef:
        name: {task name}
      params:
      - name: {image goes here}
        value: {name of image}
      - name: {args goes here}
        value: ["{list}","{of}","{arguments}","{here}"]
```

</details>

Double-check that your work matches the solution below.

### Solution

<details>
	<summary>Click here for the answer.</summary>

```yaml
    - name: lint
      workspaces:
        - name: source
          workspace: pipeline-workspace
      taskRef:
        name: eslint
      params:
      - name: image
        value: "node:18-alpine"
      - name: args
        value: ["--format", "stylish", "--ext", ".js", "."]
      runAfter:
        - clone

    # Note: The remaining tasks are unchanged
```

</details>

Apply these changes to your cluster:

```bash
kubectl apply -f pipeline.yaml
```

You should see the following output:

```
pipeline.tekton.dev/cd-pipeline configured
```

---

::page{title="Step 4: Run the pipeline"}

You are now ready to run the pipeline and see if your new lint task is working properly. You will use the Tekton CLI to do this.

Start the pipeline using the following command:

```bash
tkn pipeline start cd-pipeline \
    -p repo-url="https://github.com/ibm-developer-skills-network/ttwst-jhxyb-ci-cd-pipeline_js" \
	-p branch="main" \
    -w name=pipeline-workspace,claimName=pipelinerun-pvc \
    --showlog
```

You should see the pipeline run completely successfully. If you see errors, go back and check your work against the solutions provided.

---

::page{title="Step 5: Create a test task"}

Your pipeline also has a placeholder for a `tests` task that uses the `echo` task. Now, you will replace it with real unit tests. In this step, you will replace the `echo` task with a call to a unit test framework called `Jest`.

There are no tasks in the Tekton Catalog for `Jest`, so you will write your own.

Update the `tasks.yaml` file adding a new task called `jest` that uses the shared workspace for the pipeline and runs `npm test` in a `node:18-alpine`image.

::openFile{path="/home/project/ttwst-jhxyb-ci-cd-pipeline_js/labs/04_unit_test_automation/tasks.yaml"}

Here is a bash script to install the Node.js dependencies and run the Jest tests. You can use this as the shell script in your new task:

```
#!/bin/sh
set -e
npm ci
npm test
```

### Your task

1. Create a new task in the `tasks.yaml` file and name it `jest`. Remember, each new task must be separated using three dashes `---` on a separate line.
	<details>
		<summary>Click here for a hint.</summary>

	```yaml
	---
	apiVersion: tekton.dev/v1beta1
	kind: Task
	metadata:
	  name: {name goes here}
	```
	</details>

1. Next, you need to include the workspace that has the code that you want to test. Since Jest uses the name `source`, you can use that for consistency. Add a workspace named `source`.
	<details>
		<summary>Click here for a hint.</summary>

	```yaml
	  workspaces:
		- name: {name goes here}
	```
	</details>

1. Create a parameter called args with a description, make the type: a string, and a default: with the verbose flag \"--verbose\" as the default.
	<details>
		<summary>Click here for a hint.</summary>

	```yaml
	  params:
		- name: {name goes here}
		  description: {description goes here}
		  type: {type goes here}
		  default: "--verbose"
	```
	</details>

1. Specify the steps with one step named test that runs in a node:18-alpine image, sets workingDir as the workspace path, and uses the script above.


    <details>
		<summary>Click here for a hint.</summary>

	```yaml
	  steps:
		- name: {name goes here}
		  image: {image goes here}
		  workingDir: $(workspaces.source.path)
		  script: |
			{paste bash script here}
	```
	</details>

Double-check that your work matches the solution below.

### Solution

<details>
	<summary>Click here for the answer.</summary>

```yaml
---
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: jest
spec:
  description: This task runs Jest tests for JavaScript applications
  workspaces:
    - name: source
  params:
    - name: args
      description: Arguments to pass to Jest
      type: string
      default: "--verbose"
  steps:
    - name: test
      image: node:18-alpine
      workingDir: $(workspaces.source.path)
      script: |
        #!/bin/sh
        set -e
        npm ci
        npm test -- $(params.args)
```

</details>

Apply these changes to your cluster:

```bash
kubectl apply -f tasks.yaml
```

You should see the following output:

```
task.tekton.dev/echo configured
task.tekton.dev/cleanup configured
task.tekton.dev/eslint configured
task.tekton.dev/jest configured
```

---

::page{title="Step 6: Modify the pipeline to use Jest"}

The final step is to use the new `jest` task in your existing pipeline in place of the `echo` task placeholder.

Edit the `pipeline.yaml` file.

::openFile{path="/home/project/ttwst-jhxyb-ci-cd-pipeline_js/labs/04_unit_test_automation/pipeline.yaml"}


### Your task

Scroll down to the `tests` task definition.

1. Add a workspace named `source` that references `pipeline-workspace` to the `tests` task after the `name:` but before the `taskRef:`.
	<details>
		<summary>Click here for a hint.</summary>

	```yaml
		- name: tests
		  workspaces:
			- name: {name goes here}
			  workspace: {workspace reference goes here}
		  taskRef:
		  ...
	```
	</details>

1. Change the `taskRef:` from `echo` to reference your new `jest` task.
	<details>
		<summary>Click here for a hint.</summary>

	```yaml
		- name: tests
		  ...
		  taskRef:
			name: {name goes here}
		  ...
	```
	</details>



1. Change the `message` parameter to the `args` parameter and specify the arguments to pass to Jest as `--verbose` `--coverage`.
	<details>
		<summary>Click here for a hint.</summary>

	```yaml
		- name: {name goes here}
		  ...
		  params:
		  - name: {name goes here}
			value: "--verbose --coverage"
	```

	</details>

Double-check that your work matches the solution below.

### Solution

<details>
	<summary>Click here for the answer.</summary>

```yaml
    - name: tests
      workspaces:
        - name: source
          workspace: pipeline-workspace
      taskRef:
        name: jest
      params:
      - name: args
        value: "--verbose --coverage"
      runAfter:
        - lint
```

</details>

Apply these changes to your cluster:
```bash
kubectl apply -f pipeline.yaml
```

You should see the following output:

```
pipeline.tekton.dev/cd-pipeline configured
```

---

::page{title="Step 7: Run the pipeline again"}

Now that you have your `tests` task complete, run the pipeline again using the Tekton CLI to see your new test tasks run:

```bash
tkn pipeline start cd-pipeline \
    -p repo-url="https://github.com/ibm-developer-skills-network/ttwst-jhxyb-ci-cd-pipeline_js.git" \
	-p branch="main" \
    -w name=pipeline-workspace,claimName=pipelinerun-pvc \
    --showlog
```

You can see the pipeline run status by listing the PipelineRun with:

```bash
tkn pipelinerun ls
```

You should see:

```
$ tkn pipelinerun ls
NAME                    STARTED          DURATION   STATUS
cd-pipeline-run-6jdtk   3 minutes ago    3m11s      Succeeded
cd-pipeline-run-n9plp   15 minutes ago   1m42s      Succeeded
```

You can check the logs of the last run with:

```bash
tkn pipelinerun logs --last
```

::page{title="Conclusion"}

Congratulations! You have just created custom ESLint and Jest tasks and integrated them into your Tekton pipeline for JavaScript development.

In this lab, you learned how to create custom tasks for JavaScript linting and testing. You learned how to use `ESLint` for code quality checks and Jest for unit testing. You also learned how to create your own tasks using shell scripts and how to pass parameters into your new tasks.

## Next steps

In the next lab, you will learn how to build a container image and push it to a local registry in preparation for final deployment. In the meantime, try to set up a pipeline to build an image with Tekton from one of your own code repositories.

If you are interested in continuing to learn about Kubernetes and containers, you can get your own [free Kubernetes cluster](https://www.ibm.com/cloud/container-service/?utm_source=skills_network&utm_content=in_lab_content_link&utm_id=Lab-IBM-CD0215EN-SkillsNetwork) and your own free [IBM Container Registry](https://www.ibm.com/cloud/container-registry?utm_source=skills_network&utm_content=in_lab_content_link&utm_id=Lab-IBM-CD0215EN-SkillsNetwork).

## Author(s)
Harsh Singh

<!-- ## Change Log
| Date | Version | Changed by | Change Description |
|------|--------|--------|---------|
| 2025-07-22 | 0.1 | Sapthashree | Updated the Title as per Rav's feedback |
| 2025-01-15 | 0.2 | Ritika| Updated tekton hub verbiage and commands as they are deprecated|
| 2026-02-9 | 0.3 | Vandana| Updated option 1 and option 2 for git-clone installation|
-->


## <h3 align="center"> © IBM Corporation. All rights reserved. <h3/>


