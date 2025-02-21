# ANU FNP Catalog Infrastructure

This repository contains infrastracture deployment code for the Australian National
University, First Nations Portfolio's Catalog.

https://anufncatalogue.anu.edu.au

> [!NOTE]  
> You currently need an NCI account and assignment to the fnp project in NCI to access
> the catalog.

## Dependencies

You will need to have the following tools installed:

- [uv](https://docs.astral.sh/uv/getting-started/installation/)
- [task](https://taskfile.dev/installation)
- [pnpm](https://pnpm.io/installation)

You will also need to have a VM created with SSH access enabled.

## Usage

Build the Prez-UI static files

```bash
echo 'NUXT_PUBLIC_PREZ_API_ENDPOINT=https://your-domain/api' > prez-ui/.env
task build:prez-ui
```

Set variables for Ansible by editing the [vars.yml](./deploy/vars.yml) and 
[hosts.yml](./deploy/hosts.yml) files.

and add secret variables to the vault:

```bash
echo 'your-vault-password' > deploy/ansiblepass
uv run ansible-vault create deploy/vault.yml --vault-password-file deploy/ansiblepass
```

required vault structure is:

```yaml
fuseki:
  dataset: anufncat
  username: admin
  password: admin

letsencrypt:
  email: email@example.com
```

Then deploy:

```bash
task deploy
```
