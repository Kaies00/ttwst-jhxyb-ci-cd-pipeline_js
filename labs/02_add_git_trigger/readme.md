::page{title="Lab (Option B: JavaScript): Adding GitHub Triggers"}

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-CD0215EN-SkillsNetwork/images/IDSN-logo.png" width="200">


**Estimated time needed:** 30 minutes

Welcome to this hands-on lab for **Adding GitHub Triggers**.

Running a pipeline manually has limited uses. In this lab, you will create a Tekton Trigger to cause a pipeline run from external events like changes made to a repo in GitHub.

## Learning objective

After completing this lab, you will be able to:

- Create an EventListener, a TriggerBinding and a TriggerTemplate
- State how to trigger a deployment when changes are made to GitHub

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

![Screenshot 2025-05-27 at 12.48.03 AM.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/JPMMJXzyI_GVUxvi4qVNGw/Screenshot%202025-05-27%20at%2012-48-03%E2%80%AFAM.png)

## Change to the labs directory

Once you have cloned the repository, change to the labs directory.

```bash
cd ttwst-jhxyb-ci-cd-pipeline_js/labs/02_add_git_trigger/
```
## Navigate to the lab folder

Navigate to the `labs/02_add_git_trigger` folder in left explorer panel. All of your work will be with the files in this folder.

![Screenshot 2025-05-27 at 12.53.11 AM.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/9m3ItQ1-CFdrDc70_ZemQw/Screenshot%202025-05-27%20at%2012-53-11%E2%80%AFAM.png)

You are now ready to start the lab.

### Optional

If working in the terminal becomes difficult because the command prompt is very long, you can shorten the prompt using the following command:

```bash
export PS1="[\[\033[01;32m\]\u\[\033[00m\]: \[\033[01;34m\]\W\[\033[00m\]]\$ "
```

---

::page{title="Prerequisites"}

This lab starts with the `cd-pipeline` pipeline and `checkout` and `echo` tasks from the previous lab.

If you did not complete the previous lab, you should apply them to your Kubernetes cluster before starting this lab:

Issue the following commands to install everything from the previous labs.

```bash
kubectl apply -f tasks.yaml
kubectl apply -f pipeline.yaml
```

Check that the tasks were created:

```bash
tkn task ls
```

You should see output similar to this:

```
NAME       DESCRIPTION   AGE
checkout                 41 seconds ago
echo                     41 seconds ago
```

Check that the pipeline was created:

```bash
tkn pipeline ls
```

You should see output similar to this:

```
NAME          AGE              LAST RUN   STARTED   DURATION   STATUS
cd-pipeline   44 seconds ago   ---        ---       ---        ---
```

You are now ready to continue with this lab.

---

::page{title="Step 1: Create an EventListener"}

The first thing you need is an EventListener that is listening for incoming events from GitHub.

You will update the `eventlistener.yaml` file to define an `EventListener` named `cd-listener` that references a TriggerBinding named `cd-binding` and a TriggerTemplate named `cd-template`.

::openFile{path="/home/project/ttwst-jhxyb-ci-cd-pipeline_js/labs/02_add_git_trigger/eventlistener.yaml"}

It should initially look like this:

```yaml
apiVersion: triggers.tekton.dev/v1beta1
kind: EventListener
metadata:
  name: <place-name-here>
spec:
```

### Your Task

1. The first thing you want to do is give the EventListener a good name. Change `<place-name-Here>` to `cd-listener`.

1. The next thing is to add a service account. Add a `serviceAccountName:` with a value of `pipeline` to the spec section.

1. Now you need to define the triggers. Add a `triggers:` section under `spec:`. This is where you will define the bindings and template.

1. Add a `bindings:` section under the `triggers:` section with a `ref:` to `cd-binding`. Since multiple triggers can exist, make sure you define `- bindings` as a list using the dash `-` prefix. Also since there can be multiple bindings, make sure you defne the `- ref:` with a dash `-` prefix as well.

1. Add a `template:` section at the same level as `bindings` with a `ref:` to `cd-template`.

### Hint

<details>
	<summary>Click here for a hint.</summary>

Your eventlistener.yaml file structure should mirror this replacing the values in `{}` with the actual values:

```yaml
spec:
  serviceAccountName: {service account name here}
  triggers:
    - bindings:
      - ref: {binding reference}
      template:
        ref: {template reference}
```

</details>

Double-check that your work matches the solution below.

### Solution

<details>
	<summary>Click here for the answer.</summary>

```yaml
apiVersion: triggers.tekton.dev/v1beta1
kind: EventListener
metadata:
  name: cd-listener
spec:
  serviceAccountName: pipeline
  triggers:
    - bindings:
      - ref: cd-binding
      template:
        ref: cd-template
```

</details>

Apply the EventListener resource to the cluster:

```bash
kubectl apply -f eventlistener.yaml
```

Check that it was created correctly.

```shell
tkn eventlistener ls
```

You should see a reply similar to this:

```
NAME          AGE              URL                                                                 AVAILABLE
cd-listener   14 seconds ago   http://el-cd-listener.sn-labs-harshsingh15.svc.cluster.local:8080   True
```

You will create the TriggerBinding named `cd-binding` and a TriggerTemplate named `cd-template` in the next steps.

---

::page{title="Step 2: Create a TriggerBinding"}

Next, you need a way to bind the incoming data from the event to pass on to the pipeline. To accomplish this, you use a `TriggerBinding`.

Update the `triggerbinding.yaml` file to create a TriggerBinding named `cd-binding` that takes the `body.repository.url` and `body.ref` and binds them to the parameters `repository` and `branch`, respectively.

::openFile{path="/home/project/ttwst-jhxyb-ci-cd-pipeline_js/labs/02_add_git_trigger/triggerbinding.yaml"}

It should initially look like this:

```yaml
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerBinding
metadata:
  name: <place-name-here>
spec:
```

### Your task

1. The first thing you want to do is give the TriggerBinding the same name that is referenced in the EventListener, which is `cd-binding`.

1. Next, you need to add a parameter named `repository` to the `spec:` section with a value that references `$(body.repository.url)`.

1. Finally, you need to add a parameter named `branch` to the `spec:` section with a value that references `$(body.ref)`.

### Hint

<details>
	<summary>Click here for a hint.</summary>

Your triggerbinding.yaml file structure should mirror this replacing the values in `{}` with the actual values:

```yaml
spec:
  params:
    - name: {repository parameter}
      value: $({repository url variable reference})
    - name: {branch parameter}
      value: $({branch body variable reference})
```

</details>

Double-check that your work matches the solution below.

### Solution

<details>
	<summary>Click here for the answer.</summary>

```yaml
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerBinding
metadata:
  name: cd-binding
spec:
  params:
    - name: repository
      value: $(body.repository.url)
    - name: branch
      value: $(body.ref)
```

</details>

Apply the new TriggerBinding definition to the cluster:

```bash
kubectl apply -f triggerbinding.yaml
```

---

::page{title="Step 3: Create a TriggerTemplate"}

The TriggerTemplate takes the parameters passed in from the TriggerBinding and creates a PipelineRun to start the pipeline.

Update the `triggertemplate.yaml` file to create a TriggerTemplate named `cd-template` that defines the parameters required, and create a PipelineRun that will run the `cd-pipeline` you created in the previous lab.

::openFile{path="/home/project/ttwst-jhxyb-ci-cd-pipeline_js/labs/02_add_git_trigger/triggertemplate.yaml"}

It should initially look like this:

```yaml
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerTemplate
metadata:
  name: <place-name-here>
spec:
  params:
  # Add parameters here
  resourcetemplates:
    - apiVersion: tekton.dev/v1beta1
      kind: PipelineRun
      metadata:
        generateName: cd-pipeline-run-
      spec:
      # Add pipeline definition here
```

### Your task

You must update the parameter section of the TriggerTemplate and fill out the resourcetemplates section:

#### Update name and add parameters

1. The first thing you want to do is give the TriggerTemplate the same name that is referenced in the EventListener, which is `cd-template`.

1. Next, you need to add a parameter named `repository` to the `spec:` section with a `description:` of _\"The git repo\"_ and a `default:` of `" "`.

1. Then, you need to add a parameter named `branch` to the `spec:` section with a `description:` of _\"the branch for the git repo\"_ and a `default:` of `master`.

### Hint 1

<details>
	<summary>Click here for a hint.</summary>

The `params:` section of your triggertemplate.yaml file structure should mirror this replacing the values in `{}` with the actual values:

```yaml
spec:
  params:
    - name: {repository parameter here}
      description: {repository description}
      default: " "
    - name: {branch parameter here}
      description: {branch description}
      default: {master branch}
```
</details>

#### Complete the resource template

Finish filling out the `resourcetemplates:` section by adding the following after the commented line `# Add pipeline definition here`.

1. Add a `serviceAccountName:` with a value of `pipeline`.

1. Add a `pipelineRef:` that refers to the `cd-pipeline` created in the last lab.

1. Add a parameter named `repo-url` with a value referencing the TriggerTemplate `repository` parameter above.

1. Add a second parameter named `branch` with a value referencing the TriggerTemplate`branch` parameter above.

### Hint 2

<details>
	<summary>Click here for a hint.</summary>

The `resourcetemplates.spec:` section of your triggertemplate.yaml file structure should mirror this replacing the values in `{}` with the actual values:

```yaml
spec:
  resourcetemplates:
      spec:
	    # Add pipeline definition here
        serviceAccountName: {sa name goes here}
        pipelineRef:
          name: {pipeline name goes here}
        params:
          - name: {repository url parameter goes here}
            value: $(tt.params.repository)
          - name: {branch parameter goes here}
            value: $(tt.params.branch)
```

</details>

Double-check that your work matches the solution below.

### Solution

<details>
	<summary>Click here for the answer.</summary>

```yaml
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerTemplate
metadata:
  name: cd-template
spec:
  params:
    - name: repository
      description: The git repo
      default: " "
    - name: branch
      description: the branch for the git repo
      default: master
  resourcetemplates:
    - apiVersion: tekton.dev/v1beta1
      kind: PipelineRun
      metadata:
        generateName: cd-pipeline-run-
      spec:
        serviceAccountName: pipeline
        pipelineRef:
          name: cd-pipeline
        params:
          - name: repo-url
            value: $(tt.params.repository)
          - name: branch
            value: $(tt.params.branch)
```

</details>

> Note that while the parameter you bound from the event is a`repository`, you pass it on as `repo-url` to the pipeline. This is to show that the names do not have to match, allowing you to use any pipeline to map parameters into.

Apply the new TriggerTemplate definition to the cluster:

```bash
kubectl apply -f triggertemplate.yaml
```

---

::page{title="Step 4: Start a PipelineRun"}

Now it is time to call the event listener and start a PipelineRun. You can do this locally using the `curl` command to test that it works.

For this last step, you will need two terminal sessions. 

### Terminal 1

In one of the sessions, you need to run the `kubectl port-forward` command to forward the port for the event listener so that you can call it on `localhost`.

Use the `kubectl port-forward` command to forward port `8090` to `8080`.

```bash
kubectl port-forward service/el-cd-listener  8090:8080
```

You will see the following output, but you will not get your cursor back.

```
Forwarding from 127.0.0.1:8090 -> 8080
Forwarding from [::1]:8090 -> 8080
```

### Terminal 2

Now you are ready to trigger the event listener by posting to the endpoint that it is listening on. You will need to open a second terminal shell to issue commands.

1. Open a new Terminal shell wtih the menu item `Terminal > New Terminal`.

1. Use the `curl` command to send a payload to the event listener service.

```bash
curl -X POST http://localhost:8090 \
  -H 'Content-Type: application/json' \
  -d '{"ref":"main","repository":{"url":"https://github.com/ibm-developer-skills-network/ttwst-jhxyb-ci-cd-pipeline_js"}}'
```

This should start a PipelineRun. You can check on the status with this command:

```bash
tkn pipelinerun ls
```

You should see something like this come back:

```
NAME                    STARTED          DURATION   STATUS
cd-pipeline-run-lzxth   25 seconds ago   ---        Running
```

You can also examine the PipelineRun logs using this command (the -L means \"last\" so that you do not have to look up the name for the last run):

```bash
tkn pipelinerun logs --last
```

You should see:

```
[clone : checkout] Cloning into 'ttwst-jhxyb-ci-cd-pipeline_js'...

[lint : echo-message] Calling ESLint linter...

[tests : echo-message] Running unit tests with Jest...

[build : echo-message] Building image for https://github.com/ibm-developer-skills-network/ttwst-jhxyb-ci-cd-pipeline_js ...

[deploy : echo-message] Deploying main branch of https://github.com/ibm-developer-skills-network/ttwst-jhxyb-ci-cd-pipeline_js ...

```

---

::page{title="Conclusion"}

Congratulations, you have successfully set up Tekton Triggers.

In this lab, you learned how to create a Tekton Trigger to cause a pipeline run from external events like changes made to a repo in GitHub. You learned how to create EventListerners, TriggerTemplates, TriggerBindings, and how to start a Pipeline Run on a port.

## Next Steps

Now that you know your triggers are working, you can expose the event listener service with an ingress and call it from a webhook in GitHub and have it run on changes to your GitHub repository.

## Author(s)
Harsh

<!-- ## Change Log
| Date | Version | Changed by | Change Description |
|------|--------|--------|---------|
| 2025-05-27 | 0.1 | Harsh| Initial version created |
| 2025-05-30 | 0.2 | Pornima More | QC pass with edits |
| 2025-07-22 | 0.3 | Sapthashree | Updated the Title as per Rav's feedback |-->

## <h3 align="center"> © IBM Corporation. All rights reserved. <h3/>


