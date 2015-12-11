# Azure Resource Manager QuickStart Templates

## Template index

A searchable template index is maintained at https://azure.microsoft.com/en-us/documentation/templates/

## Contribution guide

This is a repo that contains all the currently available Azure Resource Manager templates contributed by the community. These templates are indexed on Azure.com and available to view here http://azure.microsoft.com/en-us/documentation/templates/

To make sure your template is added to Azure.com index, please follow these guidelines. Any templates that are out of compliance will be added to the **blacklist** and not be indexed on Azure.com

## Files, folders and naming conventions

1. Every template must be contained in its own folder. Name this folder something that describes what your template does. Usually this naming pattern looks like **appName-osName** or **level-platformCapability** (e.g. 101-vm-user-image) 
..* **Protip** - Try to keep the name of your template folder short so that it fits inside the Github folder name column width.
2. Github uses ASCII for ordering files and folder. For consistent ordering create all files and folder in **lowercase**.
3. Include a **readme.md** file that explains how the template works. 
..* Guidelines on the readme.md file below.
4. The template file must be named **azuredeploy.json**.
5. There should be be a parameters file name **azuredeploy.parameters.json**. 
..* Please fill out the values for the parameters according to rules defined in the template (allowed values etc.), For parameters without rules, a simple "changeme" will do as the acomghbot only checks for sytactic correctness using the ARM Validate Template [API](https://msdn.microsoft.com/en-us/library/azure/dn790547.aspx).
6. Parameter files can be used to specify the parameters for different environments. (e.g. dev, test, production). Specify additional parameter files in the format **azuredeploy.dev.parameters.json**. Where *dev* describes the environment.
7. The template folder must contain a **metadata.json** file to allow the template to be indexed on [Azure.com](http://azure.microsoft.com/).
..* Guidelines on the metadata.json file below.
8. Create a PowerShell script to deploy the template named **azuredeploy.ps1**. Create the script based on the Microsoft Azure PowerShell module version 1 or below.
..* Guidelines on the azuredeploy.ps1 file below.
9. The **custom scripts** that are needed for successful template execution must be placed in a folder called **scripts**.
10. Linked templates must be placed in a folder called **nested**.
11. Images used in the readme.md must be placed in a folder called **images**.

![alt text](https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Files, folders and naming conventions")

## readme.md

The readme describes your deployment. A good description helps other community members to understand your deployment. The readme.md uses Github Flavored Markdown for formatting text. If you want to add images to your readme file, store the images in the images folder. A good readme contains the following sections.

+ Description of what the template will deploy
+ Deploy to Azure button
+ PowerShell deployment steps based on Microsoft Azure PowerShell v1.0 or later

Do not include the parameters or the variables of the deployment script. We render this on [Azure.com] (https://azure.microsoft.com/en-us/documentation/templates/) from the template. Specifying these in the readme will result in duplicate entries on [Azure.com] (https://azure.microsoft.com/en-us/documentation/templates/).

## metadata.json

Here are the required parameters for a valid metadata.json file

To be more consistent with the Visual Studio and Gallery experience we're updating the metadata.json file structure. The new structure looks like below

```JSON
{
  "itemDisplayName": "",
  "description": "",
  "summary": "",
  "githubUsername": "",
  "dateUpdated": "<e.g. 2015-12-20>"
}
```

The metadata.json file will be validated using these rules

**itemDisplayName**

+ Cannot be more than 60 characters

**description**

+ Cannot be more than 1000 characters
+ Cannot contain HTML
+ This is used for the template description on the Azure.com index template details page

**summary**

+ Cannot be more than 200 characters
+ This is shown for template description on the main Azure.com template index page

**githubUsername**

+ Username must be the same as the username of the author submitting the Pull Request
+ This is used to display template author and Github profile pic in the Azure.com index

**dateUpdated**

+ Must be in yyyy-mm-dd format.
+ The date must not be in the future to the date of the pull request

## Deployment template guidelines