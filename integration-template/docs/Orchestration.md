# Create a Orchestration Tool Integration


## Create new Tool integration

Create a new tool integration under DevOps --> Integrations --> Tool Integrations


- Create Tool integration record **sn_devops_tool_integration**
- Create Tool Integration Capability **sn_devops_tool_capability_mapping**
    - Capability as "Orchestration"
- Create Tool Integration Capabilities  **sn_devops_integration_capability**

    - Connect: Triggered when tool is created to check the provided url and credentials are correct
    - Validate: Validate to be used to check the credentials and url provided are correct
    - Discovery : Pulls information about the pipelines that are configured 
    - Notification : Process inbound event webhooks that are sent to ServiceNow instance

    All these needs to be included with a subflow which and process the provided information.
    See the existing flows for the right inputs and required responses

![Tool integration Capabilities](./diagrams/Tool%20Integration%20Capabilities.png)

### Connect and Validate

#### Connect

Connect Subflow output : 

- Connected (Boolean ): Return true/false

#### Validatee

- response (String): in JSON format


 ```js
{   "status":true,
    "statusCode":200,
    "infoMsg":"Devops tool is connected.",
    "errorMsg":"",
    "aliasId":fd_data.subflow_inputs.aliasid
}
 ```   

### Discover

discoverPayload (String): in JSON Format array of pipelines

```js

{
    "pipelines" : [
        {
            "name" : "pipelineName",
            "url" : "Pipeine Url",
            "projectName" : "",
            "id" : "piplineId",
        }
    ]
}

```

### Handel Inbound Events notification

Refer payload for different responses for different events. 
Subflow for Notification capability is not needed, if the payload maches the transformation payload.


## Payload References for Notification Events


### Pipeline Stage Start Event


POST:  "https://myinstance.service-now.com/api/sn_devops/v1/devops/tool/Orchestration?toolId= + pipelineInfo.tool.sys_id"

```js
{
    "taskExecution": {
        "toolId": pipeline.tool.sys_id,
        "buildNumber": pipeline.buildNumber,
        "nativeId": pipeline.buildNumber,
        "name": stage.name + "#" + pipeline.buildNumber,
        "id": stage.name + "#" + pipeline.buildNumber,
        "url": pipeline.pipeline_url + "/" + stage.name + "/#" + pipeline.buildNumber,
        "isMultiBranch": "false",
        "branchName": "main",
        "pipelineExecutionUrl": pipeline.pipeline_url + "/" + stage.name + "/" + pipeline.buildNumber,
        "orchestrationTaskUrl": pipeline.pipeline_url + "/" + stage.name + "/",
        "orchestrationTaskName": pipeline.orch_pipeline + "#" + stage.name,
        "result": "building",
        "startDateTime": dateTime,
        "upstreamId": stageNumber > 0 ? previousStage.name + "#" + pipeline.buildNumber : "",
        "upstreamTaskUrl": stageNumber > 0 ? pipeline.pipeline_url + "/" + previousStage.name + "/#" + pipeline.buildNumber : "",
    },
    "orchestrationTask": {
        "orchestrationTaskURL": pipeline.pipeline_url + "/" + stage.name + "/",
        "orchestrationTaskName": pipeline.orch_pipeline + "#" + stage.name,
        "branchName": "main",
        "toolId": pipeline.tool.sys_id
    }
}
        
```


### Pipeline Stage End Event


POST:  https://myinstance.service-now.com/api/sn_devops/v1/devops/tool/Orchestration?toolId= + pipelineInfo.tool.sys_id

```js
    {
        "taskExecution": {
            "toolId": pipeline.tool.sys_id,
            "buildNumber": pipeline.buildNumber,
            "nativeId": pipeline.buildNumber,
            "name": stage.name + "#" + pipeline.buildNumber,
            "id": stage.name + "#" + pipeline.buildNumber,
            "url": pipeline.pipeline_url + "/" + stage.name + "/#" + pipeline.buildNumber,
            "isMultiBranch": "false",
            "branchName": "main",
            "pipelineExecutionUrl": pipeline.pipeline_url + "/" + stage.name + "/" + pipeline.buildNumber,
            "orchestrationTaskUrl": pipeline.pipeline_url + "/" + stage.name + "/",
            "orchestrationTaskName": pipeline.orch_pipeline + "#" + stage.name,
            "result": "successful",
            "startDateTime": dateTime,
            "endDateTime": dateTime,
            "upstreamId": previousStage.name + "#" + pipeline.buildNumber ,
            "upstreamTaskUrl":  pipeline.pipeline_url + "/" + previousStage.name + "/#" + pipeline.buildNumber ,
        },
        "orchestrationTask": {
            "orchestrationTaskURL": pipeline.pipeline_url + "/" + stage.name + "/",
            "orchestrationTaskName": pipeline.orch_pipeline + "#" + stage.name,
            "branchName": "main",
            "toolId": pipeline.tool.sys_id
        }
    }

```  

### Artifact Registration for pipeline execution

POST: "api/sn_devops/devops/artifact/registration?orchestrationToolId=" + pipelineInfo.tool.sys_id,

```js
{
    "taskExecutionNumber": pipeline.buildNumber,
    "pipelineName": pipeline.orch_pipeline,
    "stageName": stage.name,
    "artifacts": [
        {
            "name": "tool-artifact",
            "repositoryName": "env0-webapp",
            "semanticVersion": "1.0.2",
            "version": "1.0.2"
        }
    ]

}
```


### Package Registration for a pipeline execution

POST: "api/sn_devops/devops/package/registration?orchestrationToolId="+ pipelineInfo.tool.sys_id,

```js    
{
    "name": "tool-tempalte-package",
    "pipelineName": pipelineInfo.orch_pipeline,
    "stageName": stage.name,
    "taskExecutionNumber": pipelineInfo.buildNumber,
    "orchestrationTask": {
        "orchestrationTaskURL": pipelineInfo.pipeline_url + "/" + stage.name + "/",
        "orchestrationTaskName": pipelineInfo.orch_pipeline + "#" + stage.name,
        "branchName": "main",
        "toolId": "pipelineInfo.tool.sys_id",
    },
    "artifacts":[
            {
            "pipelineName" : "pipelineInfo.orch_pipeline",
            "stageName" :"stageName",
            "taskExecutionNumber" : "pipelineInfo.buildNumber",
            "semanticVersion" : "artifact.semanticVersion in MAJOR.MINOR.PATCH format",
            "version" : "artifact.version" ;
        }        
    ]
}

```

### Change Control

POST: "api/sn_devops/devops/orchestration/changeControl?toolId=" + pipeline.tool.sys_id,

```js
{
    "toolId": pipeline.tool.sys_id,
    "callbackURL": "unique call back url",
    "orchestrationTaskURL": pipeline.pipeline_url + "/" + stage.name + "/",
    "orchestrationTaskName": pipeline.orch_pipeline + "#" + stage.name,
    "orchestrationTaskDetails": {
        "message": "task message",
        "triggerType": "upstream",
        "taskExecutionURL": pipeline.pipeline_url + "/" + stage.name + "/#" + pipeline.buildNumber,
        "orchestrationTaskURL": pipeline.pipeline_url + "/" + stage.name + "/",
        "orchestrationTaskName": pipeline.orch_pipeline + "#" + stage.name,
        "branchName": "main",
        "upstreamTaskExecutionURL": pipeline.pipeline_url + "/" + previousStage.name + "/#" + pipeline.buildNumber ,
        "toolId": pipeline.tool.sys_id,
    }
}
    
```        