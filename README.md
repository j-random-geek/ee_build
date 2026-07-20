# Execution Environment build playbook

This can be run on the commandline or under AAP; the behaviour is controlled by variables. The playbook uses a single hostname for all endpoints, and so is intended for use in AAP 2.5 and above only.

## Execution environment definitions

These are stored under `vars/execution_environments` as YAML files (called _environment_.yml).
NOTE that the file extension is `yml`, NOT `yaml`.

## Variables

The playbook uses the following variables to control operation; defaults are in [group_vars/all.yml](`group_vars/all.yml`).

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

