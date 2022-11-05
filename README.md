# Tag Specification

## Architecture & Terminology

A tag is made of 2 things:
- a `key`
- a `value`

Before diving deeper into tags , let's have a quick overview of what our Platform Architecture is made of.
Overall, it is built out of 2 different kinds: `Services` & `Instances`.


### Services
- a Service in the platform world is a module, a pipeline or a template that will be used to construct/launch `instances`.
- a Platform Service can be understood as a Class. The platform is responsible for providing reliable abstractions,classes.

### Instances
- An Instance is a single piece of infrastructure or code derived from a `Service`.
- In other words, the `instance` is a `service-consumer`.
- An instance can be:
    - one of your app built on top of the `platform-chart` service.
    - a MongoDB cluster.
    - a kubernetes cluster itself.

    - NB: To be Microservice/Cloud-native compliant `Instances` must be as stateless as possible.
      i.e. the same configuration code "should" topics the same instance.

### Tag Consumer,Producer

Terminology:
- The Platform & AWS Billing account will be considered `tag-consumer`.
- Anything creating a resource will be considered a `tag-producer`.

However, if I'm trying to create a new resource which is under the hood created by a Platform Service, am I even considered
as a `tag-producer`?

We need to split this `tag-producer` in at least 2 different kinds:
- `service-tag-producer`
- `instance-tag-producer`


A tag have a `key` & a `value`. Key should adhere 100% to the convention

### Joining,Splitting character terminology:

- Keys must be joined by `/`. Slash `/` specifies relation between parent (left element) to child (right element)
- Values must be joined by `:` or split by `,`.
    - Colon `:` specifies relation between parent (left element) to child (right element).
    - Comma `,` lists several unrelated elements.


### Responsibilities

#### Tag Keys

| type                  | Responsible | Accountable | Consulted | Informed |
|:----------------------|:------------|:------------|:----------|:---------|
| tag-consumer          |             | ✔️          | ✔️        | ✔️       |
| service-tag-producer  | ✔️          | ✔️          |           |          |
| instance-tag-producer | (✔️)        |             |           |          |

Meaning of:
- Responsible:
    - `service-tag-producer` should enforce Tag Keys are conform.
    - `instance-tag-producer` will become responsible if:
        - implement any miss-usage.
        - Does not use the right Platform Service. Implementing forks or self-made solutions is at your own risks.
- Accountable:
    - `tag-consumer` should provide quality documentation to the `service-tag-producer`.
    - `service-tag-producer` is not accountable for any miss-usage of `instance-tag-producer`.
- Consulted:
    - `tag-consumer` should provide quality documentation & tools to enable the `service-tag-producer` to enforce conventions.
- Informed:
    - Tag-keys referenced by `service-tag-producer` & `instance-tag-producer` are conform to the conventions.


#### Tag Values
| type                  | Responsible | Accountable | Consulted | Informed |
|:----------------------|:------------|:------------|:----------|:---------|
| tag-consumer          | ️           |             |           | ✔️       |
| service-tag-producer  |             | ✔️          | ✔️        |          |
| instance-tag-producer | ✔️          |             |           |          |

Meaning of:
- Responsible:
    - `instance-tag-producer` must provide the right tag-values.
    - `service-tag-producer` can in some occasion enforce tag-values, but is not responsible for any miss-usage.
- Accountable:
    - `service-tag-producer` must ensure the validation of the tag-values produced by the `instance-tag-producer` (`service-consumer`)
- Consulted:
    - `service-tag-producer` should provide quality documentation & tools to enable the `instance-tag-producer` to manage the tag-values
- Informed:
    - Tag-values referenced by `service-tag-producer` & `instance-tag-producer` are conform to the conventions.

The values are the responsibilities of the `tag-producer`

## Tags naming convention

### Keys

Keys are always :
- prefixed by `platform`. (Be aware `aws` key prefix is used by AWS internally & could lead to unpredictable behaviors)
- followed by a Category: e.g. tech, business, automation, security...
- followed by a Subcategory or Subfield: e.g. name, version, id...

### Technical tags for Resource Organization

    Tags are a good way to organize AWS resources in the AWS Management Console. 
    You can configure tags to be displayed with resources, and can search and filter by tag. 
    With the AWS Resource Groups service, you can create groups of AWS resources based on one or more tags or portions of tags. 
    You can also create groups based on their occurrence in an AWS CloudFormation stack. 
    Using Resource Groups and Tag Editor, you can consolidate and view data for applications that consist of multiple services, resources, and Regions in one place.


#### Instance "tech" tags

| key                                                  | example (value)                               | description (value)                                                          |
|------------------------------------------------------|-----------------------------------------------|------------------------------------------------------------------------------|
| `instance/name`                        | eks-efk                                       | Identify resources that are related to a specific application.               |
| `instance/version`                     | v2.1.0                                        | Version of the instance                                                      |
| `instance/resource/domain`             | domain_a                                      | Domain where the resource was instantiated.                                  |
| `instance/resource/name`               | persistentVolumeClaim                         | Identify individual resources of an instance.                                |
| `instance/resource/namespace`          | aws:eks:eu-central-1:domain_a:prod:monitoring | Namespace where the resource was instantiated.                               |
| `instance/resource/region`             | eu-central-1                                  | Region where the resource was instantiated.                                  |
| `instance/resource/role`               | persistence                                   | Describe the function of a particular resource of an instance.               |
| `instance/resource/stage`              | prod                                          | Stage where the resource was instantiated.                                   |
| `instance/resource/created-by/domain`  | platform                                      | Domain that created this instance/resource.                                  |
| `instance/resource/created-by/service` | argocd                                        | "Meta Service" which instantiated this resource. (e.g. auto-devops pipeline) |

#### Service "tech" Tags.

| key                                    | example (value)   | description (value)                                                                                             |
|----------------------------------------|-------------------|-----------------------------------------------------------------------------------------------------------------|
| `service/name`           | eks-storage-class | Platform service used to instantiate this specific resource.                                                    |
| `service/version`        | v0.1.0            | Version of the platform service used to construct this `instance`.                                              |
| `service/parent/name`    | eks-common        | Platform services used to construct this `service`'s `instance`. From parent to children, joined by colons `:`. |
| `service/parent/version` | v0.1.0            | Version of the parents. Respectively from parent to children, joined by colons `:`.                             |

### Tags for Business & Cost Allocation


| key                         | example (value) | description (value)                                                                                    |
|-----------------------------|-----------------|--------------------------------------------------------------------------------------------------------|
| `business/owner`   | platform        | Identify who is responsible for the instance.                                                          |
| `business/project` | monitoring      | Identify business projects that the instance supports.                                                 |
| `business/sla`     | 42              | SLA level of the business function it supports.                                                        |
| `business/tenant`  | 1a24aaf7        | Identify a specific tenant that a particular group of instances serves.                                |
| `business/unit`    | domain_a        | Identify the cost center or business unit associated with an instance (domain or other business unit). |

Difference between Domain & Organizational Unit:
- a domain is a whole business level unit:
    - e.g. `domain_a`.
    - a domain spans `stages` (& `regions` of course).
    - is an abstractions containing all staged accounts of this business level unit.
    - does not contain or manage any cloud infrastructure. It's an abstraction.
- an organizational unit is a fully qualified identifier of an AWS account:
    - e.g. `domain_a-dev`.
    - directly contains & manages cloud infrastructure.

### Tags for Automation

    Resource or service-specific tags are often used to filter resources during automation activities.
    Automation tags are used to opt in or opt out of automated tasks or to identify specific versions of resources to archive, update, or delete.
    For example, you can run automated start or stop scripts that turn off development environments during nonbusiness hours to reduce costs.
    In this scenario, Amazon Elastic Compute Cloud (Amazon EC2) instance tags are a simple way to identify instances to opt out of this action.
    For scripts that find and delete stale, out-of-date, or rolling Amazon EBS snapshots, snapshot tags can add an extra dimension of search criteria.

| key                             | example (value)         | description (value)                                                                                                                                |
|---------------------------------|-------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------|
| `automation/date-time` | Reserved for future use | Identify the date or time a resource should be started, stopped, deleted, or rotated.                                                              |
| `automation/opt-in`    | Reserved for future use | Indicate whether an instance should be included in an automated activity such as starting, stopping, or resizing.                                  |
| `automation/opt-out`   | Reserved for future use | Indicate whether an instance should be included in an automated activity such as starting, stopping, or resizing.                                  |
| `automation/security`  | Reserved for future use | Determine requirements, such as encryption or enabling of Amazon VPC flow logs; identify route tables or security groups that need extra scrutiny. |

### Tags for Concurrency matters


| key                     | example (value) | description (value)                                                                                                   |
|-------------------------|-----------------|-----------------------------------------------------------------------------------------------------------------------|
| `mutex/author` | 1a24aaf7        | Process id of the last author of a lock,release.                                                                      |
| `mutex/locked` | false           | If `locked == true`, an application should be able to stop its execution in order to let the holder release the lock. |
| `mutex/timestamp`     | 1667681752      | Timestamp of the last lock,release (in Unix epoch).                                                                   |


### Tags for Security & access control

| key                                 | example (value) | description (value)                                                                                   |
|-------------------------------------|-----------------|-------------------------------------------------------------------------------------------------------|
| `security/compliance`      |                 | An identifier for workloads that must adhere to specific compliance requirements (e.g. DB in France). |
| `security/confidentiality` |                 | An identifier for the specific data confidentiality level an instance supports.                       |


## Values of the `instance/namespace` tag

The `instance/namespace` must be a "fully qualified" namespace:

| subfields | example                 | description                                      |
|-----------|-------------------------|--------------------------------------------------|
| Provider  | `aws`                   | Name of the provider.                            |
| Service   | `eks`                   | Provider's service used "behind the scene".      |
| L-0       | `eu-central-1`          | "Level `0` Namespace" of the provider's service. |
| L-1       | `domain_a`              | "Level `1` Namespace" of the provider's service. |
| L-2       | `prod`                  | "Level `2` Namespace" of the provider's service. |
| L-3       | `monitoring`            | "Level `3` Namespace" of the provider's service. |
| L-n       | `something-very-nested` | "Level `n` Namespace" of the provider's service. |


NB: Subfields must be joined with a colon because they express a level of relation.

Example:
```
aws:eks:eu-central-1:domain_a:prod:monitoring
```
