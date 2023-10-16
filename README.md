# My TerraTowns NegoDrama Project 2023 

## Semantic Versioning :mage:

This project is going utilize semantic versioning for its tagging.
[https://semver.org/](https://semver.org/)


The general format:

 **MAJOR.MINOR.PATCH**, eg. `1.0.1`

- **MAJOR** version when you make incompatible API changes
- **MINOR** version when you add functionality in a backward compatible manner
- **PATCH** version when you make backward compatible bug fixes

## Install Terraform CLI


### Considerations with the Terraform CLI changes
The Terraform CLI installation instructions have changed due to gpg 
keyring changes. So we needed refer to the latest install CLI 
instructions via Terraform Documentation and change the scripting
for install.

[Install Terraform CLI](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)


### Considerations for Linux distribution 


This project is built against Ubuntu.
Please consider checking your Linux Distribution and change accordingly to distribution needs.
[How to Check OS Version in Linux](https://www.cyberciti.biz/faq/how-to-check-os-version-in-linux-command-line/)

Example of Checking OS Version:
```sh
$ cat /etc/os-release
PRETTY_NAME="Ubuntu 22.04.3 LTS"
NAME="Ubuntu"
VERSION_ID="22.04"
VERSION="22.04.3 LTS (Jammy Jellyfish)"
VERSION_CODENAME=jammy
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=jammy
```

### Refactoring into Bash Scripts

While fixing the Terraform CLI gpg depreciation issues we notice 
that bash scripts steps were a considerable amount more code. 
So we decided to create a bash script to install the Terraform CLI.

This bash script is located here:[./bin/install_terraform_cli](./bin/install_terraform_cli)

- This will keep Gitpod Task File ([.gitpod.yml](./gitpod.yml)) tidy.
- This allow us an easier to debug and execute manually Terraform CLI install.
- This will allow better portablity for other projects that need to install Terraform CLI.

#### Shebang Considerations

A Shebang (pronouced Sha-bang) tells the bash script what program that will interpret the script. eg. `#!/bin/bash`

ChatGPT recommended this format for bash: `#!/usr/bin/env bash`

- for portability for different OS ditribution
- will search the user's PATH for the bash executable

https://en.wikipedia.org/wiki/Shebang_(Unix)

## Execution Considerations

When executing the bash script we can use the `./` shorthand notation to execute the bash script.

eg. `./bin/install_terraform_cli`

If we are using script in .gitpod.yml we need to point the script to a program to interpret it.

eg. `source ./bin/install_terraform_cli`

#### Linux Permissions Considerations

Linux permissions works as follows:
In order to make our bash scripts executable we need to change linux permission for the fix to be executable at the user mode. 

```sh
chmod u+x ./bin/install_terraform_cli
```

Alternatively: 

```sh
chmod 744 ./bin/install_terraform_cli
```

https://en.wikipedia.org/wiki/Chmod

### Github Lifecycle (Before, Init, Command)

We need to be careful when using the Init because it will not rerun if we restart an existing workspace.

https://www.gitpod.io/docs/configure/workspaces/tasks


### Working EV Vars

We can list out all Environment Variable (Env Vars) using the `env` command

We can filter specific env vars using greg eg. `env | grep AWS_`

#### Setting and Unsetting Env Vars

In the terminal we can set using `export HELLO`

We can set an evn var temporarily when just running a command

```sh
HELLO='world' ./bin/print_message
```
Within a bash script we can set env without writing export eg.


```sh
#!/usr/bin/env bash

HELLO='world'

echo $HELLO
```

#### Printing Vars

We can print an env var using echo eg. `echo $HELOO`

#### Scoping of Env Vars

When you open up new bash terminals in VSCode it will not be aware of env vars that you have set in another window.

If you want to Env Vars to persist across all future bash terminals that are open you need to set env vars in your bash profile. eg. `.bash_profile`

#### Persisting Env Vars in Gitpod 
We can persist env vars into gitpod by storing them in Gitpod Secrets Storage.

```sh
gp env HELLO='world'
```

All future workspaces launched will set the env vars for all bash terminal opened in thoes workspaces.

You can also set en vars in the `.gitpod.yml` but this can only contain non-senstive env vars.

### AWS CLI INSTALLATION

AWS CLI is installed for the project via the bash script 
[`./bin/install_aws_cli`](./bin/install_aws_cli)


[Geeting Started Install (AWS CLI)](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
[AWS CLI Env Vars](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html)

We can check if our AWS credentials is configured correctly by running the following AWS CLI command:
```sh
aws sts get-caller-identity
```
If it is successful you should see a json payload return that looks like this:

```json
{
    "UserId": "AIDA4L7MTPGUAED75IAQZ",
    "Account": "850362857896",
    "Arn": "arn:aws:iam::850362857896:user/negrodrama"
}
```

We'll need to generate AWS CLI credentials from IAM User in order to the user AWS CLI.


## Terraform Basics

### Terraform Registry

Terraform sources their providers and modules from the Terraform registry which located at [registry terraform.io](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/string)

- **Providers** is an interface to APIs that will allow to create resources in terraform.
- **Modules** are a way to make large amount of therraform code modular, portable and sharable.

[Random Terraform Provider](https://registry.terraform.io/providers/hashicorp/random)

### Terraform Console

We can see a list of all the terraform commands by simply typing `terraform`

#### Terraform Init

At the start of a new terraform project we will run `terraform init` to download the binaries for the terraform providers that we'll use in this project.

#### Terraform Plan

`terraform plan`

This will generate out a changeset, about the state of our infrastructure and what will be changed.

We can output this changeset ie. "plan" to passed to and apply, but often you can just ignore outputting.

#### Terraform Apply

`terraform apply`

This will run a plan and pass the changeset to be execute by terraform. Apply should proompt yes or no.

If we want to automatically approve an apply we can provide the auto approve flag eg.  `terraform apply --auto-approve`

#### Terraform Destroy

`terraform destroy`
This will destroy resources.

You can also use the auto approve flag to skip the approve prompt.
eg. ` terraform apply --auto-approve`


#### Terraform Lock Files

`.terraform.lock.hcl` contains the locked versioning for the providers or modules that should be used with this project.

The Terraform Lock File **should be committed** to your Version Control System (VSC) eg. Github

#### Terraform State Files

`.terraform.tfstate` contain information about the current state of your infrastructure.

This file **should not be committed** to your VCS.

This file can contain sensentive data.

If you lose this file, you lose knowning the state of your infrastructure.

`.terraform.tfstate.backup` is the previous file state.

#### Terraform Directory

`.terraform` directory contains binaries of terraform providers.