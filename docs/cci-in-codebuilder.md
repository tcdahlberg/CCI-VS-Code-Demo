# Using CumulusCI in Code Builder
# Installing CumulusCI
Ensure pip path is working
```bash
python -m ensurepip --default-pip
```

Upgrade pip
```bash
python -m pip install --upgrade pip
```

Install Pipx
```bash
python -m pip install --user pipx
```

Install CumulusCI using Pipx
```bash
python -m pipx install cumulusci
```

# Starting a new project
Using the command pallet in Code Builder (Ctrl+Shift+P) create a new SFDX project. You will put your new project in the default dx-project directory.
```base
SFDX: Create Project
```

Initialize git in the new project directory
```bash
git init
```

Initialize CumulusCI in the new project directory
```bash
cci project init
```

Authenticate a Dev Hub org with Code Builder using the command pallet (Ctrl+Shift+P)
```bash
  sf org login web --set-default-dev-hub --alias my-hub-org --instance-url https://the.my.domain.of.your.org
```
