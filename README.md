# Tekton Slack notifications.

As of writing this slack or any type of native notification system are not supported, so we have to make our own.

Currently the architecture is plan is below.
![](diagram.png)

When a CD pipeline is ran it sends a cloud event on termination with a status of Succeeded/Failed/Canceled 

We create a data sink to take in and digest the status of the Run and output an action that we want. In this case, a slack notification to a channel with the catalog [Slack task](https://github.com/tektoncd/catalog/tree/master/task/send-to-webhook-slack/0.1)

In order to tell Tekton to start sending those CloudEvents we must enable it by setting a EventListener url to the `default-cloud-events-sink` configMap value

The EventListener [here](build-status-el.yaml). This includes a listener for every start, fail, and success on taskRuns.

When one of these cloud events runs and triggers one of three TriggerTemplates. 
* [Started](started-status-trigger-template.yaml)
* [Failed](failed-status-trigger-template.yaml)
* [Succeeded](success-status-trigger-template.yaml)

This then executes the catalog slack tasks that sends the status to your slack channel.

Each trigger in the EventListener also requires two ref-bindings 
```yaml
apiVersion: triggers.tekton.dev/v1alpha1
kind: TriggerBinding
metadata:
  name: configmap-deploy-details
spec:
  params:
  - name: gitResource
    value: $(body.taskRun.spec.resources.inputs[?(@.name == 'source')].resourceRef.name)
---
apiVersion: triggers.tekton.dev/v1alpha1
kind: TriggerBinding
metadata:
  name: taskrun-meta
spec:
  params:
  - name: taskRunName
    value: $(body.taskRun.metadata.name)
  - name: taskRunNamespace
    value: $(body.taskRun.metadata.namespace)

```


### Types of Cloud Events

If you want to watch for different types of statuses of task or pipeline runs, reference [here](https://github.com/tektoncd/pipeline/blob/master/docs/install.md#configuring-cloudevents-notifications)

