# ACI as Code Bootcamp, Lab 3 - Use Terraform AAC Manually

## Goal

Get familar with Terraform AAC and be able to run it manually against the sandbox ACI Simulator.


## Lab Overview and Credentials

You will be able to access the APIC on the following address:

- Sandbox ACI simulator: https://sandboxapicdc.cisco.com

The ACI simulator can be accessed using the following credentials.

|            | Username | Password |
| ---------- | -------- | -------- |
| ACI Sim    | admin    | !v3G@!4@Y |


## Examine the code
Navigate to the 2_aac_diy directory:

```sh
~/prompt> cd 2_aac_diy dir
```

This folder contains a demo ACI as Code inventory (in the data dir), a main.tf file to run with Terraform and sevaral other important AaC related files (schema, rules, tests).

Open up the folder in your favorite IDE (e.g. Visual Studio, PyCharm...). Take some time and examine the code structure. All changes described in the next section will be made on the local copy of this repository.

## Customize Terraform backend

The `terraform-aac` repository we are using is pre-configured to use Terraform Cloud to store the statefile. As we will be using a local statefile in this lab, must the `main.tf` file edited and have the following `cloud` block section removed to to revert to local state storage.

```hcl
  cloud {
    organization = "balnovak"

    workspaces {
      name = "techtorial-demo"
    }
  }
```

Once this section is removed, then commit and push the file to Git.

```sh
~/prompt> git add .
~/prompt> git commit -m "Remove TF Cloud config"
```

## Install required software packages

Before Terraform AAC can be executed is it required to install a few Python packages.

```sh
~/prompt> pip install --upgrade pip
~/prompt> pip install -r requirements.txt
```

The python packages we just installed will be used to run the pre- and post-deployment validations in the next steps.

## Terraform Initialization

In addition to install the Python packages in the previous step, we also need to initialize the Terraform working directory, which includes installing the required Terraform modules and providers. This is done using the `terraform init` command.

```sh
~/prompt>  terraform init
```
