# My first github release page

## Installation Guide

Terraform will **create/ read/update/delete** resource on JDCloud automatically.Following guides show you how to install Terraform together with JDCloud plugin. Following commands are given under **Ubuntu**

#### Create a directory for your Terraform binary and JDCloud plugin

```text
mkdir terraform
cd terraform
```

#### Download and unzip Terraform binary

```text
wget releases.hashicorp.com/terraform/0.11.10/terraform_0.11.10_linux_amd64.zip
unzip terraform_0.11.10_linux_amd64.zip
```

#### Add Terraform binary into your `PATH`

```text
export PATH="$PATH:/path/to/dir"
source .profile
```

#### Download JDCloud plugin 

```text
wget github.com/XiaohanLiang/terraform-provider-jdcloud/releases/download/v0.1-beta/terraform-provider-jdcloud
```

#### Initialize Terraform with JDCloud plugin

```text
terraform init
```

#### Edit your configuration file on JDCloud. Examples on how to edit this file was shown below

```text
vim jdcloud.tf
```

## Tutorial

### Provide access key and secret key

Before everything starts you have to provide a pair of [access key](https://docs.jdcloud.com/cn/account-management/accesskey-management). 

{% code-tabs %}
{% code-tabs-item title="jdcloud.tf" %}
```text
provider "jdcloud" {
  access_key = "E1AD46FF7994BC3DF"
  secret_key = "B527396D788ABCDEF"
  region = "cn-north-1"
}
Create a VPC through Terraform
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Create a VPC resource through Terraform

VPC resource can be created by specifying the name of this VPC resource and the CIDR block. Meanwhile description on this resource is optional. Edit `jdcloud.tf` and then execute `terraform apply`. Resource on the cloud will be modified

{% code-tabs %}
{% code-tabs-item title="jdcloud.tf" %}
```text
resource "jdcloud_vpc" "vpc-1" {
  vpc_name = "my-vpc-1"
  cidr_block = "172.16.0.0/20"
  description = "example"
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Modify resource attributes through Terraform

 Just like creating them through console on web page. You can modify some attributes of resource after it has been created. Execute `terraform apply`after it has been modified

{% code-tabs %}
{% code-tabs-item title="jdcloud.tf" %}
```text
resource "jdcloud_vpc" "vpc-1" {
  vpc_name = "my-vpc-1"
  cidr_block = "172.16.0.0/20"
  description = "new and modified description"
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## More

**More example** on how to create a resource can be found [here](https://github.com/XiaohanLiang/terraform-provider-jdcloud/blob/v0.1-beta/example/main.tf).  
**Restrictions** on creating/updating a resource can be found [here](https://docs.jdcloud.com/cn/).  
**Terraform official** web page can be found [here](https://www.terraform.io/intro/index.html).

