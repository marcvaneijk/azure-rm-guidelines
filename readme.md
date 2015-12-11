# Azure Resource Manager QuickStart Templates

## Template index

A searchable template index is maintained at https://azure.microsoft.com/en-us/documentation/templates/

## Contribution guide

This is a repo that contains all the currently available Azure Resource Manager templates contributed by the community. These templates are indexed on Azure.com and available to view here http://azure.microsoft.com/en-us/documentation/templates/

To make sure your template is added to Azure.com index, please follow these guidelines. Any templates that are out of compliance will be added to the **blacklist** and not be indexed on Azure.com

## Files, folders and naming conventions

1. Every template must be contained in its own folder. Name this folder something that describes what your template does. Usually this naming pattern looks like **appName-osName** or **level-platformCapability** (e.g. 101-vm-user-image) 
	* **Protip** - Try to keep the name of your template folder short so that it fits inside the Github folder name column width.
2. Github uses ASCII for ordering files and folder. For consistent ordering create all files and folder in **lowercase**.
3. Include a **readme.md** file that explains how the template works. 
	* Guidelines on the readme.md file below.
4. The template file must be named **azuredeploy.json**.
5. There should be be a parameters file name **azuredeploy.parameters.json**. 
	* Please fill out the values for the parameters according to rules defined in the template (allowed values etc.), For parameters without rules, a simple "changeme" will do as the acomghbot only checks for sytactic correctness using the ARM Validate Template [API](https://msdn.microsoft.com/en-us/library/azure/dn790547.aspx).
6. Parameter files can be used to specify the parameters for different environments. (e.g. dev, test, production). Specify additional parameter files in the format **azuredeploy.dev.parameters.json**. Where *dev* describes the environment.
7. The template folder must contain a **metadata.json** file to allow the template to be indexed on [Azure.com](http://azure.microsoft.com/).
	* Guidelines on the metadata.json file below.
8. Create a PowerShell script to deploy the template named **azuredeploy.ps1**. Create the script based on the Microsoft Azure PowerShell module version 1 or below.
	* Guidelines on the azuredeploy.ps1 file below.
9. The **custom scripts** that are needed for successful template execution must be placed in a folder called **scripts**.
10. Linked templates must be placed in a folder called **nested**.
11. Images used in the readme.md must be placed in a folder called **images**.

![alt text](https://raw.githubusercontent.com/marcvaneijk/guidelines/master/images/filesFolderAndNamingconventions.png "Files, folders and naming conventions")

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

The following guidelines are relevant to the main deployment templates and nested templates (if used).

1. Template parameters should follow **camelCasing**
2. Try to reduce the **number of parameters** a user has to enter to deploy your template. Make things that do not need to be globally unique such as VNETs, NICs, PublicIPs, Subnets, NSGs as variables. 
	* If you must include a parameter, please include a default value as well. See the next rule for naming convention for the default values.
3. Name **variables** using this scheme **templateScenarioResourceName** (e.g. simpleLinuxVMVNET, userRoutesNSG, elasticsearchPublicIP etc.) that describe the scenario rather. This ensures when a user browses all the resources in the Portal there aren't a bunch of resources with the same name (e.g. myVNET, myPublicIP, myNSG)
4. Every parameter in the template must have the **lower-case description** tag specified using the metadata property. This looks like below

	```JSON
"newStorageAccountName": {
      "type": "string",
      "metadata": {
          "description": "The name of the new storage account created to store the VMs disks"
      }
}
	```

5. **Storage account names** need to be lower case and can't contain hyphens (-) in addition to other domain name restrictions. They also need to be globally unique. This should be configured with two variables (using the expressions **toLower** and **uniqueString**). A storageaccount has a limit of 24 characters. Set the maxLength of 11 to the parameter for the newStorageAccountName. This allows the uniqueString of 13 characters to be appended.

	```JSON
"parameters": {
    "storageAccountname": {
      "type": "string",
      "maxLength": 11,
      "metadata": {
        "description": "Name of the new storage account. Maximum Length 11 alphabetic and/or numeric characters."
      }
    }
  },
  "variables": {
      "newStorageAccountNameToLower": "[toLower(parameters('storageAccountName'))]",
      "newStorageAccountNameUnique": "[concat(variables('newStorageAccountNameToLower'), uniqueString(resourceGroup().id))]"
  },
	```

6. Every resource in the template must have the lower-case **comments** property specified.

	```JSON
      "name": "[variables('newStorageAccountNameUnique')]",
      "type": "Microsoft.Storage/storageAccounts",
      "location": "[resourceGroup().location]",
      "apiVersion": "2015-06-15",
      **"comments": "This storage account is used to store the VM disks"**
	```

7. Do not use a parameter to specify the **location**. Use the location property of the resourceGroup instead. By using the **resourceGroup().location** expression for all your resources, the resources in the template will automatically be deployed in the same location as the resource group.

	```JSON
"name": "[variables('newStorageAccountNameUnique')]",
"type": "Microsoft.Storage/storageAccounts",
**"location": "[resourceGroup().location]"**,
"apiVersion": "2015-06-15",
"comments": "This storage account is used to store the VM disks"
	```

8. Create a parameter to specify the storage namespace. Set the default value of the parameter to **core.windows.net**. Additional endpoints can be specified in the allowed value property. 

	```JSON
    "storageNamespace": {
      "type": "string",
      "defaultValue": "core.windows.net",
      "allowedValues": [
        "core.windows.net",
        "local.domain.tld"
      ],
      "metadata": {
        "description": "The endpoint namespace for storage"
      }
	```
	Create a variable that concatenates the storageAccountname and the namespace to a URI.

	```JSON
"diskUri":"[concat('http://',variables('newStorageAccountNameUnique'),'.blob.'parameters(' storageEndpoint'),'/',variables('vmStorageAccountContainerName'),'/',variables('OSDiskName'),'.vhd')]"
	```

9. Create a parameter to specify the **keyVault** namespace. Set the default value of the parameter to **vault.azure.net**. Additional endpoints can be specified in the allowed value property. 

	```JSON
      "keyVaultNamespace": {
        "type": "string",
        "defaultValue": "vault.azure.net",
        "allowedValues": [
          "vault.azure.net",
          "vault.domain.tld"
        ],
        "metadata": {
          "description": "The endpoint namespace for storage"
        }
      }
	```

	Create a variable that concatenates the key vault name and the namespace to a URI.

	```JSON
"keyVaultUri": "[concat('https://',parameters('keyVaultName'),'.'parameters(' keyVaultNamespace'))]"
	```

10. Dependencies between resources can be defined with the expression **dependsOn**. This creates an explicit dependecy. The expression **reference()** can be used to create an implicit dependency. The guidance is to use the reference() to avoid the risk having an unnecessary dependsOn element stop the deployment engine from doing aspects of the deployment in parallel. Reference can only be used against a single resource ([syntax](https://azure.microsoft.com/en-us/documentation/articles/resource-group-template-functions/#reference)), and is used for cases where the properties of one resource are needed for the provisioning of another resource. For example; a virtual machine just needs the resourceId (not properties) in this case dependsOn should be used.
11. Use **tags** to add metadata to resources for billing detail purposes. Tags should not be used to provide metadata that you will use to identify and query links between resources.

## Single template or nested templates

It is obvious to create a single deployment template for deploying a single resource. Nested templates are more common for more advanced scenarios. The following guidance helps to decide between a single template or a decomposed nested template design. 

+ Create a single template for a single tier application
+ Create a nested templates deployment for a multitier application
+ Use nested templates for conditional deployment

## Conditional nested templates

It is possible to deploy a nested template based on parameter input. The parameter input is used concatenated a path to a nested template. Based on the user input a different template is deployed. This enabled a conditional nested template deployment. Store all nested templates in the **nested** folder. The paramater is used to define the name of the template. Ensure the **allowedValues** of the input paramater match the names of the nested templates.

```JSON
    "parameters": {
        "newOrExisting": {
            "type": "String",
            "allowedValues": [
                "new",
                "existing"
            ]
        }
    },
    "variables": {
      "templateBaseUrl": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-create-availability-set",
      "templateLink": "[concat(variables('templateBaseUrl'),'/nested/',parameters('newOrExisting'),'.json')]"
    }
```

## Nested templates design for more advanced scenarios

When you decide to decompose your template design into multiple nested templates, the following guidelines will help to standardize the design.  These guidelines are based on the [best practices for designing Azure Resource Manager templates](https://azure.microsoft.com/en-us/documentation/articles/best-practices-resource-manager-design-templates/) documentation.

For this guidance a deployment of a SharePoint farm is used as an example. The SharePoint farm consists of multiple tiers. Each tier can be created with high availability. The recommended design consists of the following templates.

+ **Main template** (azuredeploy.json). Used for the input parameters.
+ **Shared resouces template**. Deploys the shared resources that all other resources use (e.g. virtual network, availability sets). The expression dependsOn enforces that this template is deployed before the other templates.
+ **Optional resources template**. Conditionally deploys resources based on a parameter (e.g. a jumpbox)
+ **Member resources templates**. Each within an application tier within has its own configuration. Within a tier different instance types can be defined. (e.g. first instance creates a new cluster, additional instances are added to the existing cluster). Each instance type will have its own deployment template.
+ **Scripts**. Widely reusable scripts are applicable for each instance type (e.g. initialize and format additional disks). Custom scripts are created for specific customization purpose are different per instance type.

![alt text](https://raw.githubusercontent.com/marcvaneijk/guidelines/master/images/nestedTemplateDesign.png "Nested templates design")

The **main template** is stored in the **root** of the folder, the **other templates** are stored in the **nested** folder. The scripts are stored in the **scripts** folder.

See the starter template [here](https://github.com/Azure/azure-quickstart-templates/tree/master/100-STARTER-TEMPLATE-with-VALIDATION) for more information on passing validation.

## Best practices

+ It is a good practice to pass your template through a JSON linter to remove extraneous commas, paranthesis, brackets that may break the "Deploy to Azure" experience. Try http://jsonlint.com/ or a linter package for your favorite editing environment (Atom, Sublime Text, Visual Studio etc.)
+ It's also a good idea to format your JSON for better readability. You can use a JSON formatter package for your local editor or [format online using this link](https://www.bing.com/search?q=json+formatter).

Starter template

A starter template is provided [here](https://github.com/Azure/azure-quickstart-templates/tree/master/100-starter-template-with-validation) for you to follow.

## Common errors from acomghbot

acomghbot is a bot designed to enforce the above rules and check the syntactic correctness of the template using the ARM Validate Template [API](https://msdn.microsoft.com/en-us/library/azure/dn790547.aspx). Below are some of the more cryptic error messages you might receive from the bot and how to solve these issues.

+ This error is received when the parameters file contains a parameter that is not defined in the template.

	The file azuredeploy.json is not valid. Response from ARM API: BadRequest - {"error":{"code":"InvalidTemplate","message":"Deployment template validation failed: 'The template parameters 'vmDnsName' are not valid; they are not present in the original template and can therefore not be provided at deployment time. The only supported parameters for this template are 'newStorageAccountName, adminUsername, adminPassword, dnsNameForPublicIP, windowsOSVersion, sizeOfDiskInGB'.'."}}

+ This error is received when a parameter in the parameter file has an empty value.

	The file azuredeploy.json is not valid. Response from ARM API: BadRequest - {"error":{"code":"InvalidTemplate","message":"Deployment template validation failed: 'The template resource '' at line '66' and column '6' is not valid. The name property cannot be null or empty'."}}

+ This error message is received when a value entered in the parameters file is different from the allowed values defined for the parameter in the template file.

	The file azuredeploy.json is not valid. Response from ARM API: BadRequest - {"error":{"code":"InvalidTemplate","message":"Deployment template validation failed: 'The provided value for the template parameter 'publicIPAddressType' at line '40' and column '29' is not valid.'."}}