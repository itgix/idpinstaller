# idpinstaller
Installer for the ITGix Application Development Platform


## Getting started
This is a set of automation tools that will allow you to rapidly spin up an empty ready to go environment in AWS with Kubernetes and a database. 
Read below on what you need to configure as prerequisites to be able to use it.

## Description
The purpose of the platform is to provide a ready build solution for software and product companies to start easily develop their new product and host it’s production environment with build-in security and redundancy without the need to build a platform from scratch. Build upon best practices that we’ve gathered over the years working with more than 60 companies and hundreds of different applications and development processes and strategies. 

The platform is based on Kubernetes and is planned to include a full-featured source control system with ready pipelines for building a generic container image or custom tailored pipelines for some of the popular languages like golang, Java, Php.

It’s based on open source products so there is no vendor lock-in, the users of the platform have the option to maintain it themselves

It’s build with strong security measures, based on practices used in FinTech for payment providers and banks. It’s ready to pass SOC  or PCI-DSS security certification.

The main platform is using AWS cloud.



## Quickstart guide

### 1. Prerequisites: You'll need to create a set of git repositories for the automation to bootstrap your environment configuration and token to access them.
- Repository to host the generated terraform code
- Repository to host the generated Kubernetes manifests that install the services on the cluster
- Optional 3rd repository to store the manifests of your custom applications that run on top of the container platform


### 2. Next, you have to run the appopriate script for your operating system to install all command line tools and prerequisites:

 This needs to be executed only once to setup your host. It will download automatically all tools and repositories needed for environment provisioning.

   - For MacOS:
   ```
    chmod +x prepare-mac.sh
    ./prepare-mac.sh
   ```
   - For Linux:
   ```
    chmod +x prepare-host.sh
    ./prepare-host.sh
   ```

### 3. Setup AWS profile:

Generate an API KEY in AWS and store it under
~/.aws/credentials
```
[myaws]
aws_access_key_id = AKIxxxxxx
aws_secret_access_key = xxxxxx
```
Setup the main profile file where you choose your main region:
~/.aws/profile
```
[profile myaws]
region=eu-central-1
```

Or use ```aws configure --profile myaws``` for a wizzard to set those up.


Note: For SSO login read below.


### 4. Prepare AWS account
If you'll use dynamic domain registration features make sure that there is a working Route53 zoen in the AWS account you provision in. Setup the zone ID and main domain in the config file.

### 5. Configuration
Use the config generator at https://adp.itgix.com or write your yaml config by modifying the template file in config/template.yaml. There is a description of the configuration options below.
Store under a directory on your local machine where you have access to like the home directory ~/myconfig.yaml

### 5. Run the command to provision your environment
   You can run in dry-run mode to only see what the installer will do without actually creating resources. You'll need to create a state bucket first if it's the first time you're running with this config.
   ```
   ./idpinstall.py --awsprofile myaws --config-file ~/myconfig.yaml --create-state-bucket-only
   ./idpinstall.py --awsprofile myaws --config-file ~/myconfig.yaml --dry-run
   ```
   - Or you can run the script in order to create the resources, without the ``` --dry-run ``` flag:
   ```
   ./idpinstall.py --awsprofile myaws --config-file ~/myconfig.yaml 
   ```
   - In that time you can check the generated logs in the logs directory of the idp-installer repository.

## 5. Work with your newly created environment

   - After the script finishes, you'll have to set your kubeconfig with the desired aks cluster - name of the cluster and region:
   ```
   aws eks update-kubeconfig list-clusters -region eu-central-1
   aws eks update-kubeconfig --name eks-ec1-dev-mysetup -region eu-central-1
   ```
   - You can check the created ingresses like this.  It takes 5-10 minutes to be created:
   ```
   kubectl get ing -A | grep argocd
   ```
   - And you can get the generated admin user password for argocd :
   ```
   kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
   ```


## Installation full guide

Either run the prepare-hosts.sh script or do the following to set-up your Python virtual environment and download required modules 

```
python3 -m venv venv
source ./venv/bin/activate
pip3 install --upgrade pip
pip3 install -r requirements.txt
deactivate

mkdir temp; mkdir git

./idpinstaller.py --awsprofile=itgixlab --dry-run

```

set access key and secret in ~/.aws/credentials

## Requirements
To be able to run the script and bootstrap the application development platform you'll need to following prepared in advance

- AWS_PROFILE - with either credentials and pofile configured in .aws/config or an assumed role with Admin priviledges for the target AWS account
- Destination git repositories with SSH key allowed to access 
  - create repository for terraform code and configure it in the config ""
  - create application repository for argo CD in advance and configure it in the config "" with the https link
    - create an access token for this repository and specify it in the config. Needed rights: read_repository and developer role.
- Linux and python3.8 installed on the machine on which you'll run this


## Usage

Command line options:
Mandatory arguments:
```
  --awsprofile AWSPROFILE
                        the AWS profile name that is configured on the local machine typically under ~/.aws/config that 
                        info about the credentials to authenticate against AWS.
```
Optional arguments:
```
  -h, --help            show this help message and exit
  --loglevel LOGLEVEL   Loglevel. warn, normal, critical. Default normal
  --dry-run             Run in dry-run mode without making actual changes
  --config-file CONFIG_FILE
                        Path to the config file. Default is config/template.yml
  --create-state-bucket-only Only creates the tfstate s3 bucket if it does not exists and then exits. 
  --update-infra        Update (overwrite) infrastructure (terraform) repository
  --update-gitops       Update (overwrite) GitOps (argocd) repositories
  --update-all          Update both infrastructure and GitOps repositories
```

Examples

```
# Example with default config (config/template.yml) just to do dry-run without actually changing anything
./idpinstaller.py --awsprofile=itgixlab --dry-run
```

Specify a configuration file
```
# Exqample Specify config file
./idpinstaller.py --awsprofile=itgixlab --config-file config/demo-dev.yml

```

The Dry-run flag allows to run the script without making actual changes to the environemnt. This allows you to test certain config, check the Terraform plan 
before actually applying or check the generated files for the destination repositories before they're comitted and pushed


When using SSO you need to first configure the profile and login, then execute normally pointing to that profile:

```commandline
## in ~/.aws/config
[profile itgixlandingzone]
region=eu-central-1
sso_start_url=https://d-9067f9b701.awsapps.com/start
sso_region=us-east-1
sso_account_id=905418051897
sso_role_name=AdministratorAccess

## Make sure you unset any old AWS keys you might have in your bash profile
$ unset AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY
## export the configured profile
$ export AWS_PROFILE=itgixlandingzone
$ aws sso login --profile itgixlandingzone
Attempting to automatically open the SSO authorization page in your default browser.
If the browser does not open or you wish to use a different device to authorize this request, open the following URL:

https://device.sso.us-east-1.amazonaws.com/

Then enter the code:

KJXH-KDPX
Successfully logged into Start URL: https://d-9067f9b701.awsapps.com/start
$ ./idpinstaller.py --awsprofile itgixlandingzone --config-file config/itgix-landing-zone.yml
Start script.                                                                                       
Read configuration file: config/itgix-landing-zone.yml                                              
Overriding tfvars git/template_repo/variable-template/terraform.tfvars with config/itgix-landing-zone.yml
Generating backend config git/template_repo/backends/backend.tfvars                                 
Initializing Terraform ...    
```


After the deployment is finished you can get the generated password for Argo CD in the following way:

```commandline
export KUBECONFIG=$PWD/temp/kubeconfig
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

To get also the generated URL of Argo you can do the following:

```commandline
$ kubectl get ingress -n argocd
NAME            CLASS    HOSTS                                  ADDRESS                                                       PORTS   AGE
argocd-server   <none>   igxadp-eu-west-1-argocd-dev.itgix.eu   k8s-public-a517c30797-516243070.eu-west-1.elb.amazonaws.com   80      28m

```


## Configuration
You can have multiple central environment configuration yaml files copied from the template.

Here is the set of available configuration options.

Mandatory:

| Option | default     | Description                                                                                                                                                                   |
|--------|-------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| project_name | no default  | The name of the project, it will be used in the naming convention , please keep short under 4 lett                                                                            |
| environment | no default  | Environment stage - sample - dev,stg,prd , but it can be any name , please keep under 4 letters                                                                               |
| region | no default  | The region in AWS, can be eu-central-1 , eu-west-2	, us-east-1	, or any of https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html   |
| aws_account_id | no default  | The AWS account ID  , unique number of the AWS tenant account to be used for this environment                                                                                 |
| env_template_repo || Source git repository for the environment template, this is the template fo IaC that will be used to bootstrap the current environemnt                                        | 
| env_template_repo_branch | | Branch of the template repository to be used                                                                                                                                  |
| env_git_repo | | Destination git repository that will be bootstrapped from the template and from there the terraform code will be applied.                                                     |
| gitops_template_repo | | Source git repository for the teplate of infastructure services that will run on the container platform , like service for managing ALBs, DNS , secrets, autoscaling and more |
| gitops_destination_repo | | Destination git repository that will be bootstrapped with infrastructure services, where argocd will be pointed to, should be a http/s as this is the protocol Argo expects |
| gitops_argo_access_token| | access token for the desitnation git repositories with read access that ArgoCD will use to get it's configuration |
| gitops_argo_access_token| | access token for the desitnation git repositories with read access that ArgoCD will use to get it's configuration |
| enable_karpenter| ||



Optional:

| Option                          | default                    | Description                                                                                                                                                                                                                             |
|---------------------------------|----------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| terraform_ver                   | 1.7                        | Version of the terraform binary that will be auto-downloaded and used                                                                                                                                                                   |
| applications_template_repo      | none                       | Source git repository for the teplate of custom applications that will run on the container platform. If left empty no application bootstrapping will be done                                                                           |
| applications_destination_repo   | none                       | Destination git repository that will be bootstrapped with application repo template , where argocd will be pointed to, should be a http/s as this is the protocol Argo expects. If left empty no application bootstrapping will be done |
| applications_argo_access_token  | none                       | access token for the desitnation git repositories with read access that ArgoCD will use to get it's configuration.If left empty no application bootstrapping will be done                                                               |
| eks_aws_auth_users              | user used for provisioning | providing additional IAM users to have access to Kubernetes apart from the user used for provisioning.  It should be a yaml list containing "username" and "group" as shown in the example below                                        |
| custom_secrets                  | none                       | Optionally generate random secrets in Secrets manager to be used later by applications. Samples provided below and in the template file                                                                                                 |


We can additionally override any of the exposed terraform variables as described below

Overriding terraform variables.

In order to override a terraform variable that you have a default for in the tf template you can place in the yaml file a variable with the same name.
We have access to all the variables that are in the variable template standard-env/variable-template/terraform.tfvars
Some specifics when needing lists or maps:
```
In tfvars we have
addons_versions = {
  coredns    = "v1.11.1-eksbuild.4"
  kube_proxy = "v1.29.0-eksbuild.1"
  vpc_cni    = "v1.16.0-eksbuild.1"
  ebs_csi    = "v1.27.0-eksbuild.1"
}
```

In the config yaml we are allowed to override in the following way

```
addons_version:
  kube_proxy: "some_other_version"
```

Sample for providing additional IAM users to have access to Kubernetes apart from the user used for provisioning 
```commandline
eks_aws_auth_users:
  - username: "myuser"
    groups:
      - "system:masters"
  - username: "aseconduser"
    groups:
      - "system:masters"
```

For more info on the available options please check the readme of the template terraform repository


## Support
Rocket channel
```
#ITGix-Container-platform
```
or log issues here in Github


## Roadmap
build an AI profile
Adapt profiles to be PCIDSS compliant

## Contributing
State if you are open to contributions and what your requirements are for accepting them.
For people who want to make changes to your project, it's helpful to have some documentation on how to get started. Perhaps there is a script that they should run or some environment variables that they need to set. Make these steps explicit. These instructions could also be useful to your future self.
You can also document commands to lint the code or run tests. These steps help to ensure high code quality and reduce the likelihood that the changes inadvertently break something. Having instructions for running tests is especially helpful if it requires external setup, such as starting a Selenium server for testing in a browser.

## Authors and acknowledgment
ITGix
Hristyan Tonev
Mihail Vukadinoff
Vladimir Dimitrov

## License
GNU GENERAL PUBLIC LICENSE
Version 3, 29 June 2007
