#  Tool-template scoped app

To test this, [Service Now DevOps Change Velocity](https://store.servicenow.com/sn_appstore_store.do#!/store/application/f1d62f041b3abc10d6f254a5624bcbf5/1.38.0?referer=%2Fstore%2Fsearch%3Flistingtype%3Dallintegrations%25253Bancillary_app%25253Bcertified_apps%25253Bcontent%25253Bindustry_solution%25253Boem%25253Butility%25253Btemplate%26q%3Ddevops%2520change%2520velocity&sl=sh) needs to be installed


## Install scoped app

- Fork this repository to your account
- Login to ServiceNow Instance
- [Import application from source control](https://docs.servicenow.com/en-US/bundle/utah-application-development/page/build/applications/task/t_ImportAppFromSourceControl.html)


## Create Devops Tool

- Navigate to DevOps Tool "Create New"
- Provide a name {example-tool}
- Tool integraiton "Tool Template"
- Tool url "tool.example.com" , credential any dummy crednetials
- Submit

## Discover

- Discover 
- This will create tool pipelines 

## Simulate data

- Open a pipeline 
- Simulate the pipeline orchestrations

### Checking the simulation 

- Check Inbound events under DevOps Inbound Events
- Refresh Steps under pipeline, this should populate new steps
- Refresh Pipeline Execution, this will show a new pipeline execution
- Check on "Pipeline UI" to see the simulated steps 

## Enabling change Control

- Open step "ChangeControl"
- Enable Change Control Flag and fill mandatory fields
- Simulate one more execution on pipeline form
- Check Change has been created at "ChangeControl" Step

Open Pipeline UI
- Check Artifacts has been created
- Check Package has been registered

## Configuring stages and Change Control





