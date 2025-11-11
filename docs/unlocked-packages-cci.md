# Creating Unlocked Packages with Salesforce CLI and CumulusCI

This guide assumes you have a project set up with CumulusCI that will build a scratch org. This project should also be connected to a GitHub repository.

## Building an unlocked package

If you project has dependencies on other packages you will need to run this command first. It converts ambigous links to GitHub into packages ids.

```bash
cci flow run dependencies --org dev 
```

Next, create a beta version of your unlocked package. Replace `November 2025` with the desired version name for your package.

```bash
cci flow run release_unlocked_beta --org dev -o create_package_version__version_name 'November 2025'  
```

CumulusCI will create an unlocked package and receipt it out to your GitHub repository in the releases section.

When you have decided the beta package is viable for a full, production release run the following command to promote the package to full release:

```bash
cci flow run release_unlocked_production
```




