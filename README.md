# pc-toolbox

A library of scripts using Prisma Cloud APIs.

## Changes

Version: 4.0 renames the scripts to adhere to Python module name conventions.

Version: 3.0 implements changes to command-line parameters; specify `-h / --help` for documentation of each script's parameters.

Version: 2.0 implements `pc_lib_api` as a PrismaCloudAPI class/instance; refer to `pcs_script_example.py` for an example of how to use it.

## Python

These scripts are written and tested in Python 3.x, but should be compatible with Python 2.x.
If you need to install Python, you can get more information at [Python's Home Page](https://www.python.org/) ... and you will also need [PIP](https://pypi.python.org/pypi/pip). 

These scripts require the following Python packages:

- requests

To install or check for these packages:

```
pip install requests pyyaml --upgrade
```

Or:

```
pip install -r requirements.txt
```

## Support

These scripts have been developed by Prisma Cloud SEs: they are not Supported by Palo Alto Networks.
But the maintainers will make a best-effort to address issues; and (of course) contributors are encouraged to submit pull requests.

## Usage

Each of these scripts requires the included `pc_lib` library directory in the same directory as the script itself.

For documentation of each script's parameters, specify `-h /--help`.


**pcs_configure.py**

API configuration options for these scripts can be specified each time on the command-line, or can be saved to a configuration file.

Use the `pcs_configure.py` script to save a Prisma Cloud Access Key, Secret Key, and API/UI Base URL in a configuration file.

- `-u / --username` (REQUIRED) Prisma Cloud Username, or Access Key generated by your Prisma Cloud User
- `-p / --password` (REQUIRED) Password associated with your Prisma Cloud Username, or Secret Key associated with your Access Key
- `--api`           (REQUIRED) Prisma Cloud API/UI Base URL used to access Prisma Cloud (`app*.prismacloud.*` ... or you can specify a direct `api*.prismacloud.*` URL). 
- `--api_compute`   (OPTIONAL) Prisma Cloud Compute API Base URL used to access Prisma Cloud (See Compute > Manage > System > Downloads: Path to Console). 
- `--config_file`   (OPTIONAL) File containing your Prisma Cloud API configuration settings. Default: `pc-settings.conf`

An Access Key/Secret Key is preferable to using a Username/Password.

The Access Key must be created by a Prisma Cloud User with the permissions required by the script(s) being executed.

Access Keys:

https://docs.paloaltonetworks.com/prisma/prisma-cloud/prisma-cloud-admin/manage-prisma-cloud-administrators/create-access-keys.html

Permissions:

https://docs.paloaltonetworks.com/prisma/prisma-cloud/prisma-cloud-admin/manage-prisma-cloud-administrators/prisma-cloud-admin-permissions.html

Compute:

The `--api_compute` parameter is required for scripts (such as `pcs_images_packages_read.py`) that use the Prisma Cloud Compute API.

For On-Premise/Self Hosted Prisma Cloud Compute, `--username` is your Prisma Cloud Compute User, and `--password` is your Password or your active Bearer token. --api/apiBase should be "" (empty string).

Settings are stored in a cleartext JSON file, by default in the same directory as the scripts themselves.
You can specify an alternate configuration file using the `--config_file` parameter.

Examples:

```
python pcs_configure.py --username "Example Access Key" --password "Example Secret Key" --api "app.prismacloud.io"
python pcs_configure.py --username "Example Access Key" --password "Example Secret Key" --api "app.prismacloud.io" --config_file ~/example-pc-settings.conf
```

Run `pcs_configure.py` specifying nothing, other than the optional (`--config_file`), to view your current configuration.


**pcs_policy_set_status.py**

Use this script to enable or disable Policies globally for an account (filtered by Policy Type or Compliance Standard).
This is primarily used to set up a new environment with every Policy enabled, or to update an environment after a large number of new Policies have been released.

Example:

```
python pcs_policy_set_status.py --policy_type config enable
```

Use this script to enable Policies that are associated with a specific Compliance Standard (or Compliance Standards).

Example:

```
python pcs_policy_status.py --policy_type all disable
python pcs_policy_status.py --compliance_standard "GDPR" enable
```

**pcs_user_import.py**

Use this script to import a list of Users from a CSV file, assigning the imported Users to the specified Role.
It will check for duplicates before importing.

Example:

```
python pcs_user_import.py "example-import-users.csv" "Example Prisma Cloud Role to assign to the imported Users"
```


**pcs_policy_custom_export.py**

Use this script to export custom Policies to a file, for backup ... or to import into another tenant.

Example:

```
python pcs_policy_custom_export.py "example-custom-policies.json"
```

**pcs_policy_custom_import.py**

Use this to script import custom Policies.
By default, imported Policies will be disabled, to maintain the status of imported Policies, specify `--maintain_status`
It will check for duplicates before importing.

Example:

```
python pcs_policy_custom_import.py "example-custom-policies.json"
```


**pcs_compliance_export.py**

Use this script to export an existing Compliance Standard (and its Requirements and Sections) to a file, for backup ... or to import into another tenant.

Example:

```
python pcs_compliance_export.py "GDPR" "example-compliance-standard.json"
```


**pcs_compliance_import.py**

Use this to script import an exported Compliance Standard (and its Requirements and Sections) into a new Compliance Standard.
To import the associated Policy mappings, specify the `--policy` parameter. 
To associate custom Policies requires first running `pcs_policy_custom_export.py` to generate a mapping file (and `pcs_policy_custom_import.py` when importing into another tenant).
It will check for duplicates before importing.

Example:

```
python pcs_compliance_import.py "example-compliance-standard.json" "GDPR Imported" --policy
```


**pcs_images_packages_read.py**

Use this script to inspect the packages in all (or one) of the container images scanned by Prisma Cloud Compute.

Example:

```
python pcs_images_packages_read.py
python pcs_images_packages_read.py --image_id "sha256:c004737361182d3cd7f38e6d9ce4a44f2a349b8dc996834e2cba0defcd0cb522"
```


**pcs_cloud_account_import_azure.py (in progress)**

This is the framework for importing a CSV (template in the `templates` directory) with a list of Azure accounts into Prisma Cloud.
Note: This is still a work in progress: the basic import framework is running, but validation of CSV and duplicate name checking has not been implemented yet.

Example:

```
python pcs_cloud_account_import_azure.py prisma_cloud_account_import_azure_template.csv
```

**pcs_posture_endpoint_client.py**

This is a generic tool for prototyping with the Cloud Security Posture API.
It sends output to stdout (and optionally to file) and errors/info sent to stderr so that it works in a pipeline which makes it 'jq' friendly.

Please note this tool is not intended as a replacement for better well-formed scripts and functions.

Example 1: GET request

```
python pcs_posture_endpoint_client.py GET /v2/policy
```

Example 2: POST request

```
cat > body.json <<EOF
{ "name": "test standard", "description":"blah" }
EOF
python pcs_posture_endpoint_client POST /compliance --request_body body.json
```

Reference: https://prisma.pan.dev/api/cloud/cspm/cspm-api

**pcs_compute_endpoint_client.py**

This is identical to `pcs_posture_endpoint_client.py`,
except it uses the Cloud Workload Protection API rather than the Cloud Security Posture API.

Reference: https://prisma.pan.dev/api/cloud/cwpp
 


