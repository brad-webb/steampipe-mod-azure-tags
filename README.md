# Azure Tags Mod for Steampipe

An Azure tags checking tool that can be used to look for untagged resources, missing tags, resources with too many tags, and more.

Run checks in a dashboard:

![image](https://raw.githubusercontent.com/turbot/steampipe-mod-azure-tags/main/docs/azure_tags_dashboard.png)

Or in a terminal:

![image](https://raw.githubusercontent.com/turbot/steampipe-mod-azure-tags/main/docs/azure_tags_mod_terminal.png)

## Getting started

### Installation

Download and install Steampipe (https://steampipe.io/downloads). Or use Brew:

```sh
brew tap turbot/tap
brew install steampipe
```

Install the Azure plugin with [Steampipe](https://steampipe.io):

```sh
steampipe plugin install azure
```

Clone:

```sh
git clone https://github.com/turbot/steampipe-mod-azure-tags.git
cd steampipe-mod-azure-tags
```

### Usage

Start your dashboard server to get started:

```sh
steampipe dashboard
```

By default, the dashboard interface will then be launched in a new browser
window at http://localhost:9194. From here, you can run benchmarks by
selecting one or searching for a specific one.

Instead of running benchmarks in a dashboard, you can also run them within your
terminal with the `steampipe check` command:

Run all benchmarks:

```sh
steampipe check all
```

Run a single benchmark:

```sh
steampipe check benchmark.untagged
```

Run a specific control:

```sh
steampipe check control.compute_virtual_machine_untagged
```

Different output formats are also available, for more information please see
[Output Formats](https://steampipe.io/docs/reference/cli/check#output-formats).

### Credentials

This mod uses the credentials configured in the [Steampipe Azure plugin](https://hub.steampipe.io/plugins/turbot/azure).

### Configuration

Several benchmarks have [input variables](https://steampipe.io/docs/using-steampipe/mod-variables) that can be configured to better match your environment and requirements. Each variable has a default defined in its source file, e.g., `controls/limit.sp`, but these can be overridden in several ways:

- Copy and rename the `steampipe.spvars.example` file to `steampipe.spvars`, and then modify the variable values inside that file
- Pass in a value on the command line:
  ```sh
  steampipe check benchmark.mandatory --var 'mandatory_tags=["Application", "Environment", "Department", "Owner"]'
  ```
- Set an environment variable:
  ```sh
  SP_VAR_mandatory_tags='["Application", "Environment", "Department", "Owner"]' steampipe check control.compute_virtual_machine_mandatory
  ```
  - Note: When using environment variables, if the variable is defined in `steampipe.spvars` or passed in through the command line, either of those will take precedence over the environment variable value. For more information on variable definition precedence, please see the link below.

These are only some of the ways you can set variables. For a full list, please see [Passing Input Variables](https://steampipe.io/docs/using-steampipe/mod-variables#passing-input-variables).

## Advanced usage

### Remediation

Using the control output and the Azure CLI, you can remediate various tagging issues.

For instance, with the results of the `compute_virtual_machine_mandatory` control, you can add missing tags with the Azure CLI:

```bash
#!/bin/bash

OLDIFS=$IFS
IFS='#'

INPUT=$(steampipe check control.compute_virtual_machine_mandatory --var 'mandatory_tags=["Application"]' --output csv --header=false --separator '#' | grep 'alarm')
[ -z "$INPUT" ] && { echo "No virtual machines in alarm, aborting"; exit 0; }

while read -r group_id title description control_id control_title control_description reason resource status resource_group subscription
do
  az tag create --resource-id ${resource} --tags Application=MyApplication
done <<< "$INPUT"

IFS=$OLDIFS
```

To remove prohibited tags from Compute virtual machines:
```bash
#!/bin/bash

OLDIFS=$IFS
IFS='#'

INPUT=$(steampipe check control.compute_virtual_machine_prohibited --var 'prohibited_tags=["Password"]' --output csv --header=false --separator '#' | grep 'alarm')
[ -z "$INPUT" ] && { echo "No virtual machines in alarm, aborting"; exit 0; }

while read -r group_id title description control_id control_title control_description reason resource status resource_group subscription
do
  az tag delete --resource-id ${resource} --name Password --yes
done <<< "$INPUT"

IFS=$OLDIFS
```

## Contributing

If you have an idea for additional controls or just want to help maintain and extend this mod ([or others](https://github.com/topics/steampipe-mod)) we would love you to join the community and start contributing.

- **[Join #steampipe on Slack →](https://turbot.com/community/join)** and hang out with other Mod developers.

Please see the [contribution guidelines](https://github.com/turbot/steampipe/blob/main/CONTRIBUTING.md) and our [code of conduct](https://github.com/turbot/steampipe/blob/main/CODE_OF_CONDUCT.md). All contributions are subject to the [Apache 2.0 open source license](https://github.com/turbot/steampipe-mod-azure-tags/blob/main/LICENSE).

Want to help but not sure where to start? Pick up one of the `help wanted` issues:

- [Steampipe](https://github.com/turbot/steampipe/labels/help%20wanted)
- [Azure Tags Mod](https://github.com/turbot/steampipe-mod-azure-tags/labels/help%20wanted)
