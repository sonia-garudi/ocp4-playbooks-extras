ocp-oadp-operator
=========

This role will:
- Install LSO (Local Storage Operator) if not already present
- Install ODF (OpenShift Data Foundation) if not already present
- Install the OADP (OpenShift APIs for Data Protection) Operator
- Run OADP e2e validation tests (Tier1) when enabled

Requirements
------------

- Running OCP 4.x cluster with 3 worker nodes is needed. Each worker node is expected to have a 500GB disk attached to it.
- Access to the cluster as a user with the cluster-admin role
- The following upstream repositories must be mirrored to your private `github.ibm.com` account before running this playbook:

| Upstream Repository | Purpose |
|---|---|
| https://gitlab.cee.redhat.com/app-mig/oadp-e2e-qe | OADP end-to-end test suite |
| https://gitlab.cee.redhat.com/app-mig/oadp-apps-deployer | Sample apps used for backup and restore testing in e2e tests |
| https://gitlab.cee.redhat.com/migrationqe/oadp-qe-automation | OADP operator install scripts |

Mirror each repository's required branch to `https://github.ibm.com/<private_github_username>/<repo-name>` and ensure the account referenced by `private_github_username` has read access.

Role Variables
--------------

| Variable | Required | Default | Comments |
|---|---|---|---|
| `oadp_enabled` | no | `false` | Set to `true` to run this playbook |
| `oadp_namespace` | no | `openshift-adp` | Namespace for the OADP operator |
| `oadp_channel` | no | `main` | Subscription channel for OADP (`stable`, `alpha`, `beta`) |
| `private_github_username` | **yes** | `""` | IBM GitHub username used to clone private repositories |
| `private_github_apikey` | **yes** | `""` | IBM GitHub API key with read access to the private repositories |
| `clone_dest` | **yes** | | Local path on the bastion where repositories will be cloned |
| `oadp_qe_automation_branch` | no | `main` | Branch to checkout from the mirrored `oadp-qe-automation` repository |
| `oadp_e2e_qe_branch` | no | `master` | Branch to checkout from the mirrored `oadp-e2e-qe` repository |
| `oadp_apps_deployer_branch` | no | `master` | Branch to checkout from the mirrored `oadp-apps-deployer` repository |
| `lso_index` | no | `""` | Catalog source index image for LSO; leave empty to use `redhat-operators` |
| `cluster_upi` | no | `true` | Set to `true` for UPI cluster, `false` for IPI |
| `volume_path` | no | `""` | Block device path for LSO (obtain via `ls /dev/disk/by-id/` on a worker node) |
| `odf_index` | no | `""` | Catalog source index image for ODF; leave empty to use `redhat-operators` |
| `odf_channel` | no | `""` | Subscription channel for ODF (e.g. `stable-4.16`); uses default channel if unset |
| `install_repository_type` | **yes** | | Passed as `REPOSITORY` env var to `deploy_oadp.sh` |
| `install_ip_approval` | **yes** | | Passed as `IP_APPROVAL` env var to `deploy_oadp.sh` |
| `install_stream` | **yes** | | Passed as `STREAM` env var to `deploy_oadp.sh` |
| `install_oadp_version` | **yes** | | Passed as `OADP_VERSION` env var to `deploy_oadp.sh` |
| `oadp_run_e2e` | no | `false` | Set to `true` to run Tier1 e2e tests after installation |
| `kubeconfig_path` | no | `/root/openstack-upi/auth/kubeconfig` | Path to kubeconfig on the bastion used for e2e tests |
| `aws_access_key_id` | **yes** (e2e) | `""` | AWS access key ID; written to `aws_creds_file` |
| `aws_secret_access_key` | **yes** (e2e) | `""` | AWS secret access key; written to `aws_creds_file` |
| `aws_creds_file` | no | `/root/aws_creds` | Path where the AWS credentials file is created on the bastion |
| `oadp_golang_tarball_url` | no | | Go tarball URL for e2e tests; auto-detected from `go.mod` if not set |
| `ginkgo_version` | no | | Ginkgo version for e2e tests; auto-detected from `go.mod` if not set |

Dependencies
------------

- `check-cluster-health` role
- `ocp-lso` role
- `ocp-odf-operator` role
- `golang-installation` role (required when `oadp_run_e2e: true`)

Example Playbook
----------------

```yaml
- name: Install OADP Operator and run validation tests
  hosts: bastion
  roles:
  - ocp-oadp-operator
```

Or with variables set inline:

```yaml
- name: Install OADP Operator and run validation tests
  hosts: bastion
  roles:
  - role: ocp-oadp-operator
    vars:
      oadp_enabled: true
      private_github_username: "my-ibm-user"
      private_github_apikey: "ghp_xxxxxxxxxxxx"
      clone_dest: "/tmp/oadp"
      oadp_qe_automation_branch: "main"
      oadp_e2e_qe_branch: "master"
      oadp_apps_deployer_branch: "master"
      volume_path: "/dev/disk/by-id/wwn-0x..."
      install_repository_type: "quay"
      install_ip_approval: "Automatic"
      install_stream: "oadp-1.4"
      install_oadp_version: "1.4.0"
      oadp_run_e2e: true
      kubeconfig_path: "/root/openstack-upi/auth/kubeconfig"
      aws_access_key_id: "AKIAIOSFODNN7EXAMPLE"
      aws_secret_access_key: "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
      ginkgo_version: "v2.17.1"
```

License
-------

See LICENCE.txt

Author Information
------------------

Sonia.Garudi@ibm.com
