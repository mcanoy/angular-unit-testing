# OpenShift Applier for Labs

https://github.com/redhat-cop/openshift-applier

# Usage

Install Requirements if not already installed.

`[.openshift-applier]$ ansible-galaxy install -r requirements.yml --roles-path=roles`

Right now limited to using ansible on your localhost.

`[.openshift-applier]$ ansible-playbook apply.yml -i inventory/`

See the inventory for the filter tag options.
