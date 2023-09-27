# Lab 2 - Use Terraform AAC Manually

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

Open up the folder in your favorite IDE (e.g. Visual Studio, PyCharm...). Take some time and examine the code structure. All changes described in the next sections will be made on the local copy of this repository.

## Change the tenant name

Since there are multiple people working on the same sandbox APIC it's necessary to separate the config scope.
In the yaml file `2_aac_diy/data/tenant_Techtorial.nac.yaml` change the tenant name to something unique, like your name.

```yaml
---
apic:
  tenants:
    - name: Techtorial-lab-BALNOVAK
...
```

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
~/prompt> terraform init   

Initializing the backend...
Initializing modules...
Downloading registry.terraform.io/netascode/nac-aci/aci 0.7.0 for aci...
- aci in .terraform/modules/aci
Downloading registry.terraform.io/netascode/aaa/aci 0.1.0 for aci.aci_aaa...
- aci.aci_aaa in .terraform/modules/aci.aci_aaa
<snip>


Initializing provider plugins...
- Finding ciscodevnet/aci versions matching ">= 2.0.0, >= 2.6.0, >= 2.6.1"...
- Finding netascode/utils versions matching ">= 0.2.5"...
- Finding hashicorp/local versions matching ">= 2.3.0"...
- Installing hashicorp/local v2.4.0...
- Installed hashicorp/local v2.4.0 (signed by HashiCorp)
- Installing ciscodevnet/aci v2.10.1...
- Installed ciscodevnet/aci v2.10.1 (signed by a HashiCorp partner, key ID 433649E2C56309DE)
- Installing netascode/utils v0.2.5...
- Installed netascode/utils v0.2.5 (self-signed, key ID 48630DA58CAFD6C0)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

## ACI as Code Validation Stage

With Terraform initialized it is time to start using ACI as Code. The first task we will perform is the pre-deployment validation, which runs a set of syntax and semantic checks against our desired configuration. This is done using a two different tools/commands:

- terraform fmt
- iac-validate

Terraform fmt verfiies the format and syntax of the Terraform scripts and can be used to either re-format the scripts on the fly or simply highligt any diviation from best practice. We will be using the latter in this lab.

```sh
~/prompt> terraform fmt -check -recursive -diff
```

If the command returns an empty output as above does it mean that no diviation from best practice was found.

The `iac-validate` tool runs the ACI as Code syntax and semantics checks as this can not be done using Terraform. `iac-validate` is a publicly available tool written by Cisco CX. In this lab are we storing a sample schema and sample validation rules withing the repository.

```sh
~/prompt> iac-validate ./data/
```

Again, if no output are provided by the tool does it mean that no syntax or scemantic issues where found.

## ACI as Code Plan Stage

With the desired configuration validated in the previous step are we ready to execute a `terraform plan`, which indicates the objects that will be added/removed from the ACI fabric in order for its configuration to match the desired configuration.

But before doing this it is required to define a number of environment variables that defines the IP address of the APIC, credentials to use, etc.

Set environment variables pointing to APIC:

```shell
export ACI_USERNAME=admin
export ACI_PASSWORD="\!v3G@\!4@Y"
export ACI_URL=https://sandboxapicdc.cisco.com
```

At this stage will no configuration changes be made to the ACI fabric, but the APIC will be queried to refresh the state entries in the Terraform statefile (if any).

```shell
~/prompt> terraform plan -out=plan.tfplan -input=false
module.aci.data.utils_yaml_merge.model: Reading...
module.aci.data.utils_yaml_merge.model: Read complete after 0s [id=977e167ae0df4b0677d92375d1fdcc44be1fad86]
module.aci.data.utils_yaml_merge.defaults: Reading...
module.aci.data.utils_yaml_merge.defaults: Read complete after 0s [id=e5d7a4194bb4e6b6fcec82da42d7b4e789871848]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # module.aci.local_sensitive_file.defaults[0] will be created
  + resource "local_sensitive_file" "defaults" {
      + content              = (sensitive value)
      + content_base64sha256 = (known after apply)
      + content_base64sha512 = (known after apply)
      + content_md5          = (known after apply)
      + content_sha1         = (known after apply)
      + content_sha256       = (known after apply)
      + content_sha512       = (known after apply)
      + directory_permission = "0700"
      + file_permission      = "0700"
      + filename             = "defaults.yaml"
      + id                   = (known after apply)
    }

<snip>

Plan: 29 to add, 0 to change, 0 to destroy.

────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Saved the plan to: plan.tfplan

To perform exactly these actions, run the following command to apply:
    terraform apply "plan.tfplan"

```

Your output may differ from the sample output above, but at the end of the output you will see a summary of the number of objects that will be added, changed, or deleted.

As we ran the terraform command with the `-out=plan.tfplan` flag was the derived plan writting to the `plan.tfplan` file. In the next section will we use this file as input when executing `terraform apply`.

## ACI as Code Apply Stage

We are now ready to apply the planned configuration changes to the ACI fabric. This will be done using the `terraform apply` command.

```sh
~/prompt> terraform apply -input=false -auto-approve plan.tfplan
module.aci.module.aci_tenant["Techtorial"].aci_rest_managed.fvTenant: Creating...
module.aci.local_sensitive_file.defaults[0]: Creating...
module.aci.local_sensitive_file.defaults[0]: Creation complete after 0s [id=e5d7a4194bb4e6b6fcec82da42d7b4e789871848]
module.aci.module.aci_tenant["Techtorial"].aci_rest_managed.fvTenant: Creation complete after 1s [id=uni/tn-Techtorial]
module.aci.module.aci_application_profile["Techtorial/DEV"].aci_rest_managed.fvAp: Creating...
module.aci.module.aci_vrf["Techtorial/VRF2"].aci_rest_managed.fvCtx: Creating...
module.aci.module.aci_vrf["Techtorial/VRF1"].aci_rest_managed.fvCtx: Creating...
module.aci.module.aci_application_profile["Techtorial/DEV"].aci_rest_managed.fvAp: Creation complete after 1s [id=uni/tn-Techtorial/ap-DEV]
module.aci.module.aci_vrf["Techtorial/VRF1"].aci_rest_managed.fvCtx: Creation complete after 1s [id=uni/tn-Techtorial/ctx-VRF1]
module.aci.module.aci_vrf["Techtorial/VRF2"].aci_rest_managed.fvCtx: Creation complete after 1s [id=uni/tn-Techtorial/ctx-VRF2]
module.aci.module.aci_vrf["Techtorial/VRF1"].aci_rest_managed.vzAny: Creating...
module.aci.module.aci_vrf["Techtorial/VRF2"].aci_rest_managed.fvRsBgpCtxPol: Creating...
module.aci.module.aci_vrf["Techtorial/VRF2"].aci_rest_managed.vzAny: Creating...
module.aci.module.aci_vrf["Techtorial/VRF1"].aci_rest_managed.fvRsBgpCtxPol: Creating...
module.aci.module.aci_vrf["Techtorial/VRF2"].aci_rest_managed.vzAny: Creation complete after 0s [id=uni/tn-Techtorial/ctx-VRF2/any]
module.aci.module.aci_vrf["Techtorial/VRF2"].aci_rest_managed.fvRsBgpCtxPol: Creation complete after 0s [id=uni/tn-Techtorial/ctx-VRF2/rsbgpCtxPol]
module.aci.module.aci_vrf["Techtorial/VRF1"].aci_rest_managed.fvRsBgpCtxPol: Creation complete after 0s [id=uni/tn-Techtorial/ctx-VRF1/rsbgpCtxPol]
module.aci.module.aci_vrf["Techtorial/VRF1"].aci_rest_managed.vzAny: Creation complete after 0s [id=uni/tn-Techtorial/ctx-VRF1/any]
module.aci.module.aci_bridge_domain["Techtorial/VLAN101"].aci_rest_managed.fvBD: Creating...
module.aci.module.aci_bridge_domain["Techtorial/VLAN102"].aci_rest_managed.fvBD: Creating...
module.aci.module.aci_bridge_domain["Techtorial/VLAN104"].aci_rest_managed.fvBD: Creating...
module.aci.module.aci_bridge_domain["Techtorial/VLAN103"].aci_rest_managed.fvBD: Creating...
module.aci.module.aci_bridge_domain["Techtorial/VLAN103"].aci_rest_managed.fvBD: Creation complete after 1s [id=uni/tn-Techtorial/BD-VLAN103]
module.aci.module.aci_bridge_domain["Techtorial/VLAN104"].aci_rest_managed.fvBD: Creation complete after 1s [id=uni/tn-Techtorial/BD-VLAN104]
module.aci.module.aci_bridge_domain["Techtorial/VLAN101"].aci_rest_managed.fvBD: Creation complete after 1s [id=uni/tn-Techtorial/BD-VLAN101]
module.aci.module.aci_bridge_domain["Techtorial/VLAN103"].aci_rest_managed.fvRsCtx: Creating...
module.aci.module.aci_bridge_domain["Techtorial/VLAN104"].aci_rest_managed.fvRsCtx: Creating...
module.aci.module.aci_bridge_domain["Techtorial/VLAN102"].aci_rest_managed.fvBD: Creation complete after 1s [id=uni/tn-Techtorial/BD-VLAN102]
module.aci.module.aci_bridge_domain["Techtorial/VLAN101"].aci_rest_managed.fvRsCtx: Creating...
module.aci.module.aci_bridge_domain["Techtorial/VLAN102"].aci_rest_managed.fvRsCtx: Creating...
module.aci.module.aci_bridge_domain["Techtorial/VLAN103"].aci_rest_managed.fvRsCtx: Creation complete after 0s [id=uni/tn-Techtorial/BD-VLAN103/rsctx]
module.aci.module.aci_bridge_domain["Techtorial/VLAN101"].aci_rest_managed.fvRsCtx: Creation complete after 0s [id=uni/tn-Techtorial/BD-VLAN101/rsctx]
module.aci.module.aci_bridge_domain["Techtorial/VLAN104"].aci_rest_managed.fvRsCtx: Creation complete after 0s [id=uni/tn-Techtorial/BD-VLAN104/rsctx]
module.aci.module.aci_bridge_domain["Techtorial/VLAN102"].aci_rest_managed.fvRsCtx: Creation complete after 0s [id=uni/tn-Techtorial/BD-VLAN102/rsctx]
module.aci.module.aci_bridge_domain["Techtorial/VLAN103"].aci_rest_managed.fvSubnet["10.1.2.1/23"]: Creating...
module.aci.module.aci_bridge_domain["Techtorial/VLAN101"].aci_rest_managed.fvSubnet["10.1.0.1/24"]: Creating...
module.aci.module.aci_bridge_domain["Techtorial/VLAN102"].aci_rest_managed.fvSubnet["10.1.1.1/24"]: Creating...
module.aci.module.aci_bridge_domain["Techtorial/VLAN104"].aci_rest_managed.fvSubnet["10.1.4.1/23"]: Creating...
module.aci.module.aci_bridge_domain["Techtorial/VLAN103"].aci_rest_managed.fvSubnet["10.1.2.1/23"]: Creation complete after 1s [id=uni/tn-Techtorial/BD-VLAN103/subnet-[10.1.2.1/23]]
module.aci.module.aci_bridge_domain["Techtorial/VLAN101"].aci_rest_managed.fvSubnet["10.1.0.1/24"]: Creation complete after 1s [id=uni/tn-Techtorial/BD-VLAN101/subnet-[10.1.0.1/24]]
module.aci.module.aci_bridge_domain["Techtorial/VLAN104"].aci_rest_managed.fvSubnet["10.1.4.1/23"]: Creation complete after 1s [id=uni/tn-Techtorial/BD-VLAN104/subnet-[10.1.4.1/23]]
module.aci.module.aci_bridge_domain["Techtorial/VLAN102"].aci_rest_managed.fvSubnet["10.1.1.1/24"]: Creation complete after 1s [id=uni/tn-Techtorial/BD-VLAN102/subnet-[10.1.1.1/24]]
module.aci.module.aci_endpoint_group["Techtorial/DEV/VLAN102"].aci_rest_managed.fvAEPg: Creating...
module.aci.module.aci_endpoint_group["Techtorial/DEV/VLAN103"].aci_rest_managed.fvAEPg: Creating...
module.aci.module.aci_endpoint_group["Techtorial/DEV/VLAN101"].aci_rest_managed.fvAEPg: Creating...
module.aci.module.aci_endpoint_group["Techtorial/DEV/VLAN104"].aci_rest_managed.fvAEPg: Creating...
module.aci.module.aci_endpoint_group["Techtorial/DEV/VLAN103"].aci_rest_managed.fvAEPg: Creation complete after 0s [id=uni/tn-Techtorial/ap-DEV/epg-VLAN103]
module.aci.module.aci_endpoint_group["Techtorial/DEV/VLAN102"].aci_rest_managed.fvAEPg: Creation complete after 0s [id=uni/tn-Techtorial/ap-DEV/epg-VLAN102]
module.aci.module.aci_endpoint_group["Techtorial/DEV/VLAN103"].aci_rest_managed.fvRsBd: Creating...
module.aci.module.aci_endpoint_group["Techtorial/DEV/VLAN101"].aci_rest_managed.fvAEPg: Creation complete after 0s [id=uni/tn-Techtorial/ap-DEV/epg-VLAN101]
module.aci.module.aci_endpoint_group["Techtorial/DEV/VLAN102"].aci_rest_managed.fvRsBd: Creating...
module.aci.module.aci_endpoint_group["Techtorial/DEV/VLAN104"].aci_rest_managed.fvAEPg: Creation complete after 0s [id=uni/tn-Techtorial/ap-DEV/epg-VLAN104]
module.aci.module.aci_endpoint_group["Techtorial/DEV/VLAN101"].aci_rest_managed.fvRsBd: Creating...
module.aci.module.aci_endpoint_group["Techtorial/DEV/VLAN104"].aci_rest_managed.fvRsBd: Creating...
module.aci.module.aci_endpoint_group["Techtorial/DEV/VLAN103"].aci_rest_managed.fvRsBd: Creation complete after 0s [id=uni/tn-Techtorial/ap-DEV/epg-VLAN103/rsbd]
module.aci.module.aci_endpoint_group["Techtorial/DEV/VLAN102"].aci_rest_managed.fvRsBd: Creation complete after 0s [id=uni/tn-Techtorial/ap-DEV/epg-VLAN102/rsbd]
module.aci.module.aci_endpoint_group["Techtorial/DEV/VLAN101"].aci_rest_managed.fvRsBd: Creation complete after 0s [id=uni/tn-Techtorial/ap-DEV/epg-VLAN101/rsbd]
module.aci.module.aci_endpoint_group["Techtorial/DEV/VLAN104"].aci_rest_managed.fvRsBd: Creation complete after 0s [id=uni/tn-Techtorial/ap-DEV/epg-VLAN104/rsbd]

Apply complete! Resources: 29 added, 0 changed, 0 destroyed.
```


Once the configuration changes have been applied to the ACI fabric, this can then be verified through using the APIC GUI.

With Terrafrom are only the configuration elements described in the desired configuration and/or present in the Terraform statefile added, changed, or removed when executing `terraform apply`.

As we in this lab are using the local filesystem for the Terraform state file `terraform.tfstate` will this file just been created/modified.

```sh
~/prompt> ls -la terraform.tfstate
-rw-r--r--  1 balnovak  staff  356033 Sep 26 12:44 terraform.tfstate
```

## ACI as Code Testing Stage

Using manual verification of configuration changes are naturally not a scalable solution, which is why automated testing is available in ACI as code.

In the Terraform flavor of AAC will we use the `iac-test` tool to render the test templates against the desired configuration and then executes the tests.

```sh
~/prompt> iac-test --data ./data --data ./defaults.yaml --templates ./tests/templates --output ./tests/results/aci |& tee test_output.txt
```

Like the tool we used for validation are the `iac-test` a publicly available tool written by Cisco CX. In this lab we use sample test templates withing the repository.

## Examining Terraform state tracking

Open the state file in your editor. You can see that it track resources in a human readable json format. Always handle this file as confidential!

Select a resource from the state file and remove it in the APIC GUI, then run `terraform apply` again. What can you see?


## Lab Summary

If you have followed the steps outline in this lab guide have you successfully used Terraform AAC to modify the configuratons of the sandbox ACI simulator.

In the next lab will we integrate these steps into a CI/CD pipeline in order to eliminate manual tasks.
