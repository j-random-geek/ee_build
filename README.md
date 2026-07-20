# Execution Environment build playbook

This can be run on the commandline or under AAP; the behaviour is controlled by variables. The playbook uses a single hostname for all endpoints, and so is intended for use in AAP 2.5 and above only.

## Execution environment definitions

These are stored under `vars/execution_environments` as YAML files (called _environment_.yml).
NOTE that the file extension is `yml`, NOT `yaml`.

## Variables

The playbook uses the following variables to control operation; defaults are in [`group_vars/all.yml`](group_vars/all.yml).

The AAP hostname, username, and password can all be supplied by a `Red Hat Ansible Automation Platform` credential in AAP. If running from the CLI, you will need to define these.

| Variable name | Type | Mandatory | Default value | Comment |
|:--------------|:-----|:---------:|:--------------|:--------|
| `ee_build_image` | String | Y | - | Which image to build. The playbook will look for `vars/execution_environments/{{ ee_build_image }}.yml` and will fail if this file does not exist. |
| `ee_create_controller_def` | Boolean | N | `false` | Whether or not to configure/update the EE definition in Controller. Leave as `false` to test the image before updating the Controller configuration. |
| `aap_hostname` | String | Y | - | The base URL of the AAP platform gateway, e.g. `https://tower.woolworths.com.au` |
| `aap_username` | String | Y | - | The username used to connect to AAP. This user needs sufficient privilege to pull images, push images, and update Execution Environment definitions in Controller. |
| `aap_password` | String | Y | - | The password for the user specified in `aap_username`. |
| `pah_api_token` | String | N | - | A Galaxy API Token for your Private Automation Hub (i.e. AAP itself). A credential using a custom Credential Type can be used for this (see below). If left undefined, the playbook will create one. Be aware that in AAP 2.5 and 2.6, this will invalidate any existing API token for the user you specify. |
| `ee_reg_credential` | String | N | `PAH Container Registry` | The AAP Credential that AAP uses to connect to the Private Automation Hub |

## Using with ansible-playbook

To use this with `ansible-playbook`, first update [`collections/requirements.yml`](collections/requirements.yml) to match your version of AAP; the file contains an uncommented section for AAP 2.5 and commented-out sections for AAP 2.6 and 2.7. To use with AAP 2.7, comment out the lines for AAP 2.5 and uncomment the lines for AAP 2.7. Then use ansible-galaxy to install the required collections:

`ansible-galaxy collection install -f -r collections/requirements.yml`

Then update [`inventory.yml`](inventory.yml) with the host that you will be using to build the execution environment image (i.e. replace `dev2.localdomain` in the inventory with the FQDN of your container build host).

Next, you will need to provide the credentials. We suggest using an vaulted variables file outside the repository, containing the following variables:

```
aap_hostname: gateway.aap.localdomain
aap_username: admin  # You can use a non-admin user but that introduces limitations on 
                     # automatic configuration of execution environments in Controller.
aap_password: 'your password'
aap_validate_certs: true  # or false, if that is required in your environment
#
# pah_api_token is optional; if not specified, one will be generated. Generating a token in 
# AAP 2.5 or 2.6 may cause issues with source control update/project sync, though, so use
# that capability with care.
#
# pah_api_token: private automation hub API token
```

Finally, run `ansible-playbook` with the appropriate arguments. To create an Execution Environment image for the
`networks` definition and **not** configure it in Controller, use the following:

`ansible-playbook -i inventory.yml build_ee.yml -e survey_which_ee=networks -e survey_controller_update=false`

To build the `custom_ee_image` EE image and configure it in Controller:

`ansible-playbook -i inventory.yml build_ee.yml -e survey_which_ee=networks -e survey_controller_update=true`


## Using with Ansible Automation Platform

To use this repository with Ansible Automation Platform
1. create a Project using this repository as the source.
2. Create an Inventory with a `builder` group, which should contain the RHEL machine you will
   use as a container build host (container-in-container builds, using underlying infrastructure such
   as OpenShift or Podman, is beyond the scope of this document).
3. Create a Credential of the type `Red Hat Ansible Automation Platform` with the base URLs
   (`https://gateway.local.domain`), username, and password of the user that will interact with
   AAP. It is easiest to use a platform administrator account, but with some fiddling you can use
   an account with lesser privilege.
4. Ensure you have a Machine credential that allows you to connect to your container build RHEL server
   with sudo access.
5. (Optional) Create a Credential Type and a Credential containing a Private Automation Hub API token
   (for AAP 2.5 or 2.6), or an OAuth2 Token (for AAP 2.7).
6. Create a Job Template using the Project from step 1, the Inventory from step 2, and the Credentials from
   steps 3, 4, and (optionally) 5.
7. (Optional) Set `pah_api_token` as an extra variable on the job template (instead of step 5 above; if neither
   is done, a new token will be generated as noted previously).
8. Add a Survey to the Job Template, prompting for:
     - the Execution Environment image to build (most foolproof is to create this as a multiple-choice,
       single-select with a list of the EE images configured in `vars/execution_environments/`), and
     - whether or not to configure the image in Controller (again. a multiple-choice, single-select list,
       with the values `true` and `false`, with `false` being the default)
9. Enable the Survey in the Job Template

You should then be able to run the job template to build a new Execution Environment image.


## Credential Type

A custom Credential Type can be used in AAP to provide the Private Automation Hub token. One suggested credential type definition is below:

```
- name: EE Build Automation Hub Token
  description: Automation Hub API Token for Execution Environment builds
  kind: cloud
  inputs:
    fields:
      - id: token
        type: string
        label: Automation Hub API Token
        secret: true
    required:
      - token
  injectors:
    extra_vars:
      pah_api_token: "{{ token }}"
```

## License

BSD 3-Clause

