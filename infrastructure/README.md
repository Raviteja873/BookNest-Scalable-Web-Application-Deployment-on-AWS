# Infrastructure

The current repository documents an AWS console-created infrastructure deployment.

Implemented resources:

- `BookNest-VPC`
- `BookNest-IGW`
- `BookNest-Public-Subnet-A`
- `BookNest-Public-Subnet-B`
- `BookNest-Public-RT`
- `BookNest-EC2-SG`
- `BookNest-ALB-SG`
- `BookNest-Server`
- `BookNest-TG`
- `BookNest-ALB`

No Terraform or CloudFormation source is included because Infrastructure as Code was not part of the implemented version.

A future version can add:

```text
infrastructure/
└── terraform/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    └── README.md
```

If Terraform is added, never commit:

```text
*.tfstate
*.tfstate.*
*.tfvars
*.tfvars.json
```

unless the files are demonstrably non-sensitive and intentionally versioned.
