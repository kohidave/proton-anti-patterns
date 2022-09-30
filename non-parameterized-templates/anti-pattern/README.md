# Non-Parameterized Template 👎

This directory contains three templates. One associated with dev, one associated with staging and one with prod. Each of these templates is almost exactly the same except a few parameters. Rather than duplicate this template three times, we should parameterize it (like in the `cool-pattern` directory). 

### Template Authors 🧑‍🏫
In this example, each template defines an AppRunner example that looks like below. The only difference in _each_ of these templates is the Image and CPU/Mem. Rather than having three templates, we could replace this parameters with jinja values (see `cool-pattern` directory).

Having a bunch of templates like this makes service management more difficult, as well as template management. 

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
            Port: '8080'
          ImageIdentifier: '1234567890.dkr.ecr.us-west-2.amazonaws.com/my-cool-app:vcda2123' # This should be parameterized
          ImageRepositoryType: ECR
      InstanceConfiguration:
        Cpu: !FindInMap [TaskSize, large, cpu]        # These should be parameterized
        Memory: !FindInMap [TaskSize, large, memory]
```

