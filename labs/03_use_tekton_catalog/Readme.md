::page{title="Lab (Option B: JavaScript): Use Tekton Continuous Delivery (CD) Catalog"}

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-CD0215EN-SkillsNetwork/images/IDSN-logo.png" width="200">


**Estimated time needed:** 30 minutes

Welcome to the hands-on lab for **Using the Tekton Catalog**. The Tekton community provides a wide selection of tasks and pipelines that you can use in your CI/CD pipelines, so that you do not have to write all of them yourself. Many commonly used tasks are available in the Tekton Catalog, which is hosted on GitHub and indexed on [Artifact Hub](https://artifacthub.io/packages/search?repo=tekton-catalog-tasks "Artifact Hub") for discovery. 


## Learning objectives

After completing this lab, you will be able to:

- Use the Tekton Catalog to install the git-clone task
- Describe the parameters required to use the git-clone task
- Use the git-clone task in a Tekton pipeline to clone your Git repository

---

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

![Screenshot 2025-05-27 at 1.33.28 AM.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/hydtUPp-uJAhplKE0K5a8A/Screenshot%202025-05-27%20at%201-33-28%E2%80%AFAM.png)

## Change to the labs directory

Once you have cloned the repository, change to the labs directory.

```bash
cd ttwst-jhxyb-ci-cd-pipeline_js/labs/03_use_tekton_catalog/
```

## Navigate to the labs folder

Navigate to the `labs/03_use_tekton_catalog` folder in the left explorer panel. All of your work will be with the files in this folder.

![Screenshot 2025-05-27 at 1.34.44 AM.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/8UoWIo6PWGyRjTz3eo67dg/Screenshot%202025-05-27%20at%201-34-44%E2%80%AFAM.png)

You are now ready to begin with the prerequisites in the next section.

### Optional

If working in the terminal becomes difficult because the command prompt is very long, you can shorten the prompt using the following command:

```bash
export PS1="[\[\033[01;32m\]\u\[\033[00m\]: \[\033[01;34m\]\W\[\033[00m\]]\$ "
```

---

::page{title="Prerequisites"}

This lab requires the installation of the tasks introduced in previous labs. To be sure, apply the previous tasks to your cluster before proceeding:

```bash
kubectl apply -f tasks.yaml
```

You should see the output similar to this:
> Note: If the tasks are already installed, the output will say \"configured\" instead of \"created.\"

```text
task.tekton.dev/echo created
task.tekton.dev/checkout created
```

You are now ready to start the lab.

---

::page{title="Step 1: Add the git-clone task"}

You start by finding a task to replace the `checkout` task you initially created. While it was OK as a learning exercise, it needs a lot more capabilities to be more robust, and it makes sense to use the community-supplied task instead.

You can browse the Tekton Catalog, find the git-clone yaml file, copy the URL to the `yaml` file, and use `kubectl` to apply it manually.

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

**Verify the installation**

After installation, confirm that the task is available:

```bash
kubectl get task git-clone

```
You should see the git-clone task listed in the output.


::page{title="Step 2: Create a workspace"}

Viewing the `git-clone` task requirements, you see that while it supports many more parameters than your original `checkout` task, it only *requires* two things:

1. The URL of a Git repo to clone, provided with the `url` param
2. A workspace called `output`

You start by creating a `PersistentVolumeClaim` (PVC) to use as the workspace:

A workspace is a disk volume that can be shared across tasks. The way to bind to volumes in Kubernetes is with `PersistentVolumeClaim`.

Since creating PVCs is beyond the scope of this lab, you have been provided with the following `pvc.yaml` file with these contents:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pipelinerun-pvc
spec:
  storageClassName: skills-network-learner
  resources:
    requests:
      storage:  1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
```

Apply the new task definition to the cluster:

```bash
kubectl apply -f pvc.yaml
```

You should see the following output:

```text
persistentvolumeclaim/pipelinerun-pvc created
```

You can now reference this persistent volume by its name `pipelinerun-pvc` when creating workspaces for your Tekton tasks.

::page{title="Step 3: Add a workspace to the pipeline"}

In this step, you will add a workspace to the pipeline using the persistent volume claim you just created. To do this, you will edit the `pipeline.yaml` file and add a `workspaces:` definition as the first line under the `spec:` but before `params:` and call it `pipeline-workspace`. Then, you will add the workspace to the pipeline `clone` task and change the task to reference `git-clone` instead of your `checkout` task.

::openFile{path="/home/project/ttwst-jhxyb-ci-cd-pipeline_js/labs/03_use_tekton_catalog/pipeline.yaml"}

### Your task

1. Edit the `pipeline.yaml` file and add a `workspaces:` definition as the first line under the `spec:` but before `params:` and call it `pipeline-workspace`.

1. Next, add the workspace to the `clone` task after the `name:` and call it `output` because this is the workspace name that the `git-clone` task will be looking for.

1. Change the name of `taskRef` in the `clone` task to reference the `git-clone` task instead of `checkout`.

1. Finally, change the name of the `repo-url` parameter to `url` because this is the name the `git-clone` task expects, but keep the mapping of `$(params.repo-url)`, which is what the pipeline expects. Also, rename the `branch` parameter to `revision`, which is what `git-clone` expects.

### Hint

<details>
	<summary>Click here for a hint.</summary>

```yaml
spec:
  workspaces:
    - name: {name goes here}
  tasks:
    - name: clone
      workspaces:
        - name: {name goes here}
          workspace: {workspace name goes here}
      taskRef:
        name: {referenced task goes here}
      params:
	  - name: {url parameter goes here}
        value: $(params.repo-url)
      - name: {revision parameter goes here}
        value: $(params.branch)

	# Note: The remaining tasks are unchanged
```

</details>

Double-check that your work matches the solution below.

### Solution

<details>
	<summary>Click here for the answer.</summary>

```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: cd-pipeline
spec:
  workspaces:
    - name: pipeline-workspace
  params:
    - name: repo-url
    - name: branch
      default: "master"
  tasks:
    - name: clone
      workspaces:
        - name: output
          workspace: pipeline-workspace
      taskRef:
        name: git-clone
      params:
      - name: url
        value: $(params.repo-url)
      - name: revision
        value: $(params.branch)

    # Note: The remaining tasks are unchanged. Do not delete them.
```

</details>

Apply the pipeline to your cluster:

```bash
kubectl apply -f pipeline.yaml
```

You should see output similar to this:
> Note: If the original pipeline was already created, you will see the word \"configured\" instead of \"created.\"

```text
pipeline.tekton.dev/cd-pipeline created
```

You are now ready to run your pipeline.

---

::page{title="Step 4: Run the pipeline"}

You can now use the Tekton CLI (`tkn`) to create `PipelineRun` to run the pipeline.

Use the following command to run the pipeline, passing in the URL of the repository, the branch to clone, the workspace name, and the persistent volume claim name.

```bash
tkn pipeline start cd-pipeline \
    -p repo-url="https://github.com/ibm-developer-skills-network/ttwst-jhxyb-ci-cd-pipeline_js.git" \
	-p branch="main" \
    -w name=pipeline-workspace,claimName=pipelinerun-pvc \
    --showlog
```

You should see output similar to this:

```text
$ tkn pipeline start cd-pipeline \
    -p repo-url="https://github.com/ibm-developer-skills-network/ttwst-jhxyb-ci-cd-pipeline_js.git" \
    -p branch="main" \
    -w name=pipeline-workspace,claimName=pipelinerun-pvc \
    --showlog
PipelineRun started: cd-pipeline-run-62q4r
Waiting for logs to be available...
```

Eventually, you should see the output from the logs.

```text
[clone : clone] + '[' false '=' true ]
[clone : clone] + '[' false '=' true ]
[clone : clone] + '[' false '=' true ]
[clone : clone] + CHECKOUT_DIR=/workspace/output/
[clone : clone] + '[' true '=' true ]
[clone : clone] + cleandir
[clone : clone] + '[' -d /workspace/output/ ]
[clone : clone] + rm -rf '/workspace/output//*'
[clone : clone] + rm -rf '/workspace/output//.[!.]*'
[clone : clone] + rm -rf '/workspace/output//..?*'
[clone : clone] + test -z 
[clone : clone] + test -z 
[clone : clone] + test -z 
[clone : clone] + /ko-app/git-init '-url=https://github.com/ibm-developer-skills-network/ttwst-jhxyb-ci-cd-pipeline_js.git' '-revision=main' '-refspec=' '-path=/workspace/output/' '-sslVerify=true' '-submodules=true' '-depth=1' '-sparseCheckoutDirectories='
[clone : clone] {"level":"info","ts":1748365778.2729099,"caller":"git/git.go:170","msg":"Successfully cloned https://github.com/ibm-developer-skills-network/ttwst-jhxyb-ci-cd-pipeline_js.git @ 0105207455eb050399ed19499fecb5cad4d88db9 (grafted, HEAD, origin/main) in path /workspace/output/"}
[clone : clone] {"level":"info","ts":1748365778.334829,"caller":"git/git.go:208","msg":"Successfully initialized and updated submodules in path /workspace/output/"}
[clone : clone] + cd /workspace/output/
[clone : clone] + git rev-parse HEAD
[clone : clone] + RESULT_SHA=0105207455eb050399ed19499fecb5cad4d88db9
[clone : clone] + EXIT_CODE=0
[clone : clone] + '[' 0 '!=' 0 ]
[clone : clone] + printf '%s' 0105207455eb050399ed19499fecb5cad4d88db9
[clone : clone] + printf '%s' https://github.com/ibm-developer-skills-network/ttwst-jhxyb-ci-cd-pipeline_js.git

[lint : echo-message] Calling ESLint linter...

[tests : echo-message] Running unit tests with Jest...

[build : echo-message] Building image for https://github.com/ibm-developer-skills-network/ttwst-jhxyb-ci-cd-pipeline_js.git ...

[deploy : echo-message] Deploying main branch of https://github.com/ibm-developer-skills-network/ttwst-jhxyb-ci-cd-pipeline_js.git ...
```

You can always see the pipeline run status by listing `PipelineRuns` with:

```bash
tkn pipelinerun ls
```

You should see:

```
NAME                    STARTED        DURATION   STATUS
cd-pipeline-run-62q4r   1 minute ago   32s        Succeeded
```

You can check the logs of the last run with:

```bash
tkn pipelinerun logs --last
```

---

::page{title="Conclusion"}

Congratulations! You have just added a task from the Tekton Catalog instead of writing it yourself. You should get into the habit of always checking the Tekton Catalog before writing any task. Remember: _\"A line of code you did not write is a line of code that you do not have to maintain!\"_

In this lab, you learned how to use the `git-clone` task from the Tekton catalog. You learned how to install the task by applying the catalog YAML to your cluster using `kubectl` and how to modify your pipeline to reference the task and configure its parameters. You also learned how to start a pipeline with the Tekton CLI `pipeline start` command and monitor its output using `--showlog`.

## Next steps

In the next lab, you will use a combination of self-written and catalog tasks to fill out your pipeline in future labs. In the meantime, try to set up a pipeline to build an image with Tekton from one of your own code repositories.

If you are interested in continuing to learn about Kubernetes and containers, you can get your own [free Kubernetes cluster](https://www.ibm.com/cloud/container-service/?utm_source=skills_network&utm_content=in_lab_content_link&utm_id=Lab-IBM-CD0215EN-SkillsNetwork) and your own free [IBM Container Registry](https://www.ibm.com/cloud/container-registry?utm_source=skills_network&utm_content=in_lab_content_link&utm_id=Lab-IBM-CD0215EN-SkillsNetwork).

## Author(s)
Harsh
<!-- ## Change Log
| Date | Version | Changed by | Change Description |
|------|--------|--------|---------|
| 2025-07-22 | 0.3     | Sapthashree | Updated the Title as per Rav's feedback |--
| 2026-01-14 | 0.4 | Ritika Joshi | Updated tekton hub verbiage and commands as they are deprecated|
| 2026-02-09 | 0.5 | Md Haroon Hussain | Updated tekton and Artifact Hub verbiage and commands |
>

## <h3 align="center"> © IBM Corporation. All rights reserved. <h3/>

