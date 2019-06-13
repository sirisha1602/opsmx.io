# Steps to operate terraform service from spinnaker pipeline stage


## prerequisite

  * Run OpsMx terraApp service on same Debian spinnaker.
  * Create custom webhook stage in spinnaker.
  * Install terraform binary on same Debian spinnaker and OpsMx terraApp service machine.

## Run opsmx terraApp service


download tar Link: <pre><code>https://github.com/OpsMx/Terraform-spinnaker/raw/master/terraApp/artifacts/zipfol.tar.gz </code></pre>
extract tar you will find terraApp.jar inside zipfol.
open terminal go inside zipfol directory and run terraApp service by the following command

 
```yaml
	nohup java -Dserver.port=8090 -jar terraApp.jar & 
```

## Create custom webhook stage in spinnaker

create file in (~/.hal/default/profile/) directory by name orca-local.yml and copy custom webhook stage block given below 
replace (ip:port) in url part "https://<ip:port>/api/v1/deleteTerraform" example http://192.x.x.135:8090/api/v1/startTerraform -- 192.x.x.135:8090 is
ip and port of spinnaker same machine on which opsmx terraApp service is running


### custom webhook stage block

```yaml
	webhook: 
  preconfigured: 

    - label: Terraform-delete-plan
      description: "For delete terraform plan"
      enabled: true
      type: terraformCWStage
      url: "http://<ip:port>/api/v1/deleteTerraform"
      method: POST

      payload: |-
          {

            "applicationName": "${execution['application']}",
            "pipelineName": "${execution['name']}",
            "pipelineId": "${execution['id']}"   
          }

    - label: Terraform-plan
      description: "For create terraform plan"
      enabled: true
      type: terraformCWStage
      url: "http://<ip:port>/api/v1/startTerraform"
      method: POST

      payload: |-
          {

            "gitAccount": "${parameterValues['gitAccount']}",
            "cloudAccount": "${parameterValues['cloudAccount']}",
            "cloudKind": "${parameterValues['cloudKind']}",
            "applicationName": "${execution['application']}",
            "pipelineName": "${execution['name']}",
            "pipelineId": "${execution['id']}",   
            "plan": "${parameterValues['plan']}"

          }


      parameters: 
        - label: "Cloud provider account"
          description: ""
          name: cloudAccount
          type: string

        - label: "Cloud provider"
          description: ""
          name: cloudKind
          type: string

        - label: "Git Account"
          description: ""
          name: gitAccount
          type: string

        - label: "Git Commit"
          description: ""
          name: plan
          type: string

```
after doing setting in orca-local.yml do hal deploy apply after deploying spinnaker successfully our setting will add two stages in spinnaker 

 1. Terraform-plan stage having four input block


     * a. Cloud provider account – Pick cloud provider account name from your spinnaker clouds accounts which we are going to use as a terraform infrastructure.
     * b. Cloud provider – Provide the Type of Cloud Provider account, for example – Kubernetes, Openshift, etc.
     * c. GitAccount – Provide the GITAccount account name which is configured in your spinnaker this mandatory when your terraform plan present on same git hub account because we are supporting terraform plan as a remote artifact as well as in plan source fill             plan details in next text box means (d) step.
     * d. Terraform plan – Provide the Terraform Plan here, we are accepting both inline and remote terraform plan
          a. Example of Remote terraform plan:<pre><code> https://github.com/OpsMx/TerraformPlansModule.git//Namespace</code></pre> (where https://github.com/OpsMx/TerraformPlansModule.git is git repo where //Namespace is one terraform module on the same repo) 
          b. Example of inline terraform plan: place the whole content of this link->
	         <pre><code>https://github.com/OpsMx/TerraformPlansModule/blob/master/Namespace/main.tf</code></pre>


  2. Terraform-delete-plan stage having no input block it used for deleting a created terraform plan.

### Now steps to experience running integrated Terraform-plan custom webhook stage from your spinnaker 

## Use-case

will create a pipeline which has two stages one- Terraform-plan and second- deploy(manifest) in Terraform-plan stage will
use for creating a namespace in kubernetes and in deploy(manifest) will deploy deployment in the same namespace which 
creates by previous Terraform-plan stage

 1. Create an Application to initiate Pipeline creation.
 2. Click on pipeline configure, add stage from the dropdown select Terraform-plan. 
        ![Screenshot](image1)
       * a. Cloud provider account - your kubernetes account name.
       * b. Cloud provider – kubernetes.
       * c. GitAccount - your git account.
	   * d. Terraform plan – https://github.com/<your-github-account>/TerraformPlansModule.git//Namespace. fork this GitHub 
	     repo <pre><code>https://github.com/OpsMx/TerraformPlansModule.git</code></pre> in your GitHub account. 
 3. Click on Add Stage, select Deploy Manifest then select the Text radio button for Manifest Source and copy deployment manifest block     given below.

      ### deployment manifest block
  ```yaml
        apiVersion: extensions/v1beta1
        kind: Deployment
        metadata:
        name: terratest
        namespace: >-
        ${#stage('Terraform-plan')['context']['buildInfo']['outputValues']['nameSpace']}
        spec:
        replicas: 2
        selector:
         matchLabels:
         app: terratest
        strategy:
        type: RollingUpdate
        template:
        metadata:
          labels:
            app: terratest
        spec:
          containers:
            - image: 'docker.io/opsmx11/restapp:v1'
              imagePullPolicy: Always
              name: restapp
              ports:
                - containerPort: 8090
                  name: http
                  protocol: TCP

 	
  ```


 4. Save pipeline and run a pipeline



