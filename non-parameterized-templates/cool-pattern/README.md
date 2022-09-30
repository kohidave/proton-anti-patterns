# Paramaterized Template

This directory includes a parameterized template. Developers can provide a custom tasksize, port and container image per environment they want to deploy to. 

### Template Authors 🧑‍🏫
Template authors define the inputs they want devs to provide in the `schema.yaml` directory - I've copied it down below. 

```yaml
schema:
  format:
    openapi: "3.0.0"
  service_input_type: "ApprunnerServiceInput"
  pipeline_input_type: "PipelineInputs"

  types:
    ApprunnerServiceInput:
      type: object
      description: "Input properties for the App runner service"
      properties:
        port:
          title: "Port"
          type: number
          description: "The port to route traffic to"
          default: 80
          minimum: 0
          maximum: 65535
        task_size:
          title: "Task size"
          type: string
          description: "The size of the task you want to run"
          enum: ["medium", "large"]
          default: "medium"
        image:
          title: "Image"
          type: string
          description: "The name/url of the container image"
          minLength: 1
          maxLength: 200

    PipelineInputs:
      type: object
      description: "Pipeline input properties"
      properties:
        service_dir:
          title: "Application Code Directory"
          type: string
          description: "Source directory for the service"
          default: "ecs-static-website"
          minLength: 1
          maxLength: 100
        dockerfile:
          title: "Dockerfile"
          type: string
          description: "The location of the Dockerfile to build"
          default: "Dockerfile"
          minLength: 1
          maxLength: 100
        unit_test_command:
          title: "Unit test command"
          type: string
          description: "The command to run to unit test the application code"
          default: "echo 'add your unit test command here'"
          minLength: 1
          maxLength: 200

```

You can see in the `instance_infrastructure/cloudformation.yaml` file that [we can use JINJA](https://docs.aws.amazon.com/proton/latest/userguide/ag-infrastructure-tmp-files.html#cloudformation) to inject this variables into the template. 

```yaml
  AppRunnerService:
    Type: AWS::AppRunner::Service
    Properties:
      SourceConfiguration:
        AuthenticationConfiguration:
          AccessRoleArn:
            Fn::GetAtt:
              - ServiceAccessRole
              - Arn
        ImageRepository:
          ImageConfiguration:
            Port: '{{service_instance.inputs.port}}' # Look, this value is defined in the schema we defined above!
          ImageIdentifier: '{{service_instance.inputs.image}}' # Look, this value is defined in the schema we defined above!
          ImageRepositoryType: ECR
      InstanceConfiguration:
        Cpu: !FindInMap [TaskSize, {{service_instance.inputs.task_size}}, cpu] # Look, this value is defined in the schema we defined above!
        Memory: !FindInMap [TaskSize, {{service_instance.inputs.task_size}}, memory] # Look, this value is defined in the schema we defined above!

```

### Developers 🤠

If a developer wanted to deploy their code using this template to a particular environment, they'd fill update the spec file to include the inputs defined in the schema. As you can see, the developer has a different container image for prod than dev and staging. They also choose to use a larger task size for prod, than dev and staging. 

The important thing is, these inputs are all associated with _one_ template. The template is parameterized that when a customer deploys to prod, the prod configuration is read from this spec, and the template is rendered using that configuration. 

```yaml
proton: ServiceSpec
pipeline:
  service_dir: .
  dockerfile: Dockerfile
  unit_test_command: echo 'add your unit test command here'
instances:
  - name: my-cool-service-dev
    environment: dev
    spec:
      port: 8080
      task_size: medium
      image: 1234567890.dkr.ecr.us-west-2.amazonaws.com/my-cool-app:vcda2123   

  - name: my-cool-service-staging
    environment: staging
    spec:
      port: 8080
      task_size: large
      image: 1234567890.dkr.ecr.us-west-2.amazonaws.com/my-cool-app:vcda2123   
      
  - name: my-cool-service-prod
    environment: prod
    spec:
      port: 8080
      task_size: large
      image: 1234567890.dkr.ecr.us-west-2.amazonaws.com/my-cool-app:bbc319sc   

```
