### Litmus Chaos Executor

-   The executor runs the litmus jobs & monitors them. Typically invoked by the litmuschaos operator to run
    experiments listed in the chaosengine
-   To learn more about the chaosengine & chaosexperiment resources, see [chaos operator](https://github.com/litmuschaos/chaos-operator)

### Table of contents

<table style="border:1 solid">
<tr>
<td>  How to override chaosexperiments env variables from chaosengine  </td>
<td> <a href="https://github.com/shubhamchaudhary/litmus/tree/update-executor-readme/executor#how-to-override-chaosexperiment-defaults"> click here </a> </td>
</tr>
<tr>
<td> How to delete completed/stale jobs after execution of experiment </td>
<td> <a href="https://github.com/shubhamchaudhary/litmus/tree/update-executor-readme/executor#how-to-delete-stalecompleted-jobs-after-chaosexperiment-execution"> click here </a> </td>
</tr>
<tr>
<td> How to override executor and exporter image from chaosengine </td>
<td> <a href="#overrideimages"> click here </a> </td>
</tr>
<tr>
<td>  Limitations </td>
<td> <a href="#limitations"> click here </a> </td>
</tr>
</table>

<div id="overrideenvs">

### How to Override ChaosExperiment defaults

-   To override the ENV variables in the chaos experiment CRs, perform one of the below steps before applying the manifest:

    -   Make changes in the ChaosExperiment CR manifest,

    ```
    apiVersion: litmuschaos.io/v1alpha1
    description:
      message: "Kills a container belonging to an application pod \n"
    kind: ChaosExperiment
    metadata:
      name: container-kill
      version: 0.1.1
    spec:
      definition:
        image: "litmuschaos/ansible-runner:ci"
        args:
          - -c
          - ansible-playbook ./experiments/generic/container_kill/container_kill_ansible_logic.yml -i
           / etc/ansible/hosts -vv; exit 0
        command:
          - /bin/bash
        env:
          - name: ANSIBLE_STDOUT_CALLBACK
            value: 'default'

          - name: TARGET_CONTAINER  <--- want to Override this env variable from chaosengine
            value: 'container-1'

          - name: KILL_MODE
            value: ''

          - name: LIB
            value: ''

        labels:
          name: container-kill
 
    ```    

-   Add those variables in the component spec of the respective experiments in the chaosEngine CR manifest

  ```
    apiVersion: litmuschaos.io/v1alpha1
    kind: ChaosEngine
    metadata:
      name: engine-nginx
      namespace: default
    spec:
      monitoring: true 
      jobCleanUpPolicy: ''
      appinfo:
        appns: default  
        applabel: app=nginx
        appkind: deployment
      chaosServiceAccount: litmus
      components:
        monitor:
          image: '' 
        runner:
          image: ''
      experiments:
      - name: container-kill
        spec:
          components:
          - name: TARGET_CONTAINER   <--- ADD HERE
            value: 'container-2'
  ```
</div>

<div id="deletejobs">

### How to Delete stale/completed jobs after chaosExperiment Execution

-   To ensure the completed experiment jobs are removed, set `spec.jobCleanupPolicy` to `delete`

  ```
    apiVersion: litmuschaos.io/v1alpha1
    kind: ChaosEngine
    metadata:
      name: engine-nginx
      namespace: default
    spec:
      monitoring: true 
      jobCleanUpPolicy: ''  <--- ADD HERE
      appinfo: 
        appns: default  
        applabel: app=nginx
        appkind: deployment
      chaosServiceAccount: litmus
      components:
        monitor:
          image: '' 
        runner:
          image: ''
      experiments:
        - name: container-kill
          spec:
            components:
            - name: TARGET_CONTAINER
              value: 'container-2'
  ```
</div>

<div id="overrideimages">

### How to Override Executor and Exporter image

-   To provide a chaos executor (runner) & exporter (monitor) of choice, provide the image names (in the form registry/user/image:tag) in the chaosEngine as shown below. The defaults at this point correspond to the litmuschaos CI images in dockerhub:
    
    -   Executor: litmuschaos/ansible-runner:ci
    -   Monitor: litmuschaos/chaos-exporter:ci

-   Make changes in the ChaosEngine CR manifest,
        
  ```
    apiVersion: litmuschaos.io/v1alpha1
    kind: ChaosEngine
    metadata:
      name: engine-nginx
      namespace: default 
    spec:
      monitoring: true 
      jobCleanUpPolicy: '' 
      appinfo: 
        appns: default  
        applabel: app=nginx
        appkind: deployment
      chaosServiceAccount: litmus
      components:
        monitor:
          image: ''     <----------- ADD EXPORTER(MONITOR) IMAGE HERE
        runner:
          image: ''     <----------- ADD EXECUTOR(RUNNER) IMAGE HERE
      chaosServiceAccount: litmus 
      experiments:
        - name: container-kill
          spec:
            components:
            - name: TARGET_CONTAINER
              value: 'container-2'
  ```
</div>

<div id="limitations">

### Limitations

-   Executor is currently unable to parse more than one configmap.

-   The name of file which contains data for configmap in experimentCR should be parameters.yml

-   The configmap is mounted in this default directory: /mnt/ 

</div>
