---
id: meshstack.aws.landing-zones
title: Landing Zones
---

AWS Landing Zones use Cloud Formation templates to orchestrate configuration of managed AWS accounts. In the following section the options for this Landing Zone are described.

You have two paths for provisionig AWS accounts. You can simply use CloudFormation StackSets in order to bootstrap your account or you can use an existing, "external" (in the sense of not controlled via meshstack) Account Vending Machine (AVM). Decide for a bootstrap path and configure the Landing Zone accordingly.

## Parameters

Each Landing Zone has paremeters which control its behavior. The available parameters are described below.

### Target Organization Unit ID

This parameter is optional. If it is set all meshProjects placed under this Landing Zone will be put under this Organization Unit (OU). This might be helpful if a SCP should be assigned to all of these projects. If left empty a new OU will be created for every customer and all of his meshProjects with AWS location will be placed in it.

You must provide the ID of the OU which can be found in the AWS Organizational Management panel. The IDs start with `ou-*`.

You can also use a root account under which the accounts will be placed when they are created. Root IDs start with `r-*`.

### AccessStack Template URL

This StackSet will be created in every new AWS account. It usually contains simple bootstraping resources e.g. like required roles and drops the rights of the admin access which is pre-assigned to each new account by AWS (the `OrganizationAccountAccessRole`).

### StackSet ARN

Operators can also configure an [CloudFormation StackSet](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/what-is-cfnstacksets.html) on an AWS Landing Zone. meshStack will then ensure that all AWS Accounts provisioned using this Landing Zone will receive a StackInstance from the StackSet. This allows operators to leverage that StackSet to centrally manage configuration and resources for all AWS Accounts under the landing zone.

Each AWS project which now gets this Landing Zone assigned will be setup to receive the Cloud Formation Stack Instance setup.

The following parameters can be used in the StackSet template:

| Parameter    | Description                                                                              |
| ------------ | :--------------------------------------------------------------------------------------- |
| AccountName  | Account name of the created AWS account                                                  |
| AccountEmail | Account email of the created AWS account                                                 |
| ProductName  | Customer Identifier                                                                      |
| CostCenter   | ID of the CostCenter defined for this meshProject.                                       |
| ContactEmail | E-Mail of the meshProject creator. Currently this is the user which created the project. |
| Stage        | The project identifier                                                                   |

> Soon the parameter names will change but will be editable in the Landing Zone configuration.

#### StackInstance Deploy Region

The StackInstances will be deployed in this region.

> **Important:** A StackInstance can currently only be deployed in one region. In order to work around you can create another StackInstance in different regions triggered by the first instance.

### Lambda ARN

As some users might have already their own custom AWS account provisioning solution, usually called Account Vending Machines (AVM), other mechanisms are needed in order to bootstrap newly created AWS accounts.

While it is possible to trigger an AVM execution via a StackSet a better alternative is usually to trigger it by an custom Lambda function invocation. During the AWS account [replication](./meshcloud.tenant.md) meshStack is able to invoke such a function with custom parameters. From this function the external AVM can start its provisioning process.

> Please make sure the `MeshfedServiceRole` has the rights in order to trigger the configured Lambda.


The following parameters can be used in the Lambda function and are provided as a JSON payload like the following pattern:

```json
{
  "AccountName": "max-muster",
  "AccountEmail": "max@muster.de",
  // ...
}
```

| Parameter    | Description                                                                              |
| ------------ | :--------------------------------------------------------------------------------------- |
| AccountName  | Account name of the created AWS account                                                  |
| AccountEmail | Account email of the created AWS account                                                 |
| ProductName  | Customer Identifier                                                                      |
| CostCenter   | ID of the CostCenter defined for this meshProject.                                       |
| ContactEmail | E-Mail of the meshProject creator. Currently this is the user which created the project. |
| Stage        | The project identifier                                                                   |

## Account Vending Machines

In order to detect the finished execution of the AVM meshStack looks for certain tags on the AWS account. If such a tag is present the success of the external [replication](./meshcloud.tenant.md) is assumed and the [replication](./meshcloud.tenant.md) process is handed back to meshStack.

Currently the account must contain these tags to be picked up:

* ProductName: Must contain the customer identifier
* Stage: Must contain the project identifier

> These tags will soon be customizable in the AWS Landing Zone configuration.

Please contact meshcloud for more details and reference configurations.
