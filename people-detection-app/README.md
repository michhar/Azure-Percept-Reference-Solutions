# People Counting reference solution
			
## Overview

This is an open-source Percept DK reference solution providing edge-based people counting with user-defined zone entry/exit events. Video and AI output from the on-prem edge device is egressed to Azure Data Lake using Azure Media Service and IoT Hub, with the user interface running as an Azure Website. AI inferencing is provided by Azure Percept DK for people detection:

![People Detector](docs/images/People-Detector-AI.gif)

## Physical hardware app topology

### Device Requirements
This application requires an Azure Percept DK.

![People Detector](docs/images/azure-percept-device.png)

# Installation Details
This open-source reference solution showcases best practices for AI security, privacy and compliance.  It is intended to be immediately useful for anyone to use with their Percept DK device. 

## Prerequisites
- A service principal with an object id, app id, and secret is required in order to deploy the solution. Please ensure you have the appropriate rights to create such a service principal, or contact your Azure Active Directory Administrator to assist you in creating one. You can follow this [how-to guide](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal) to create the required service principal.

- You must have owner access on the Azure subscription you wish to deploy to

- To install this application you must have Owner level access to the target subscription.  This deployment will create resources within the following Azure namespaces. These resource providers must be enabled on the subscription.
    * Microsoft.Devices
    * Microsoft.Authorization
    * Microsoft.ContainerInstance
    * Microsoft.ManagedIdentity
    * Microsoft.Web
    * Microsoft.Storage
    * Microsoft.Resources
    * Microsoft.Media

- Azure Percept DK with Vision SOM attached

- You should have already completed the [Azure Percept DK Setup Experience](https://docs.microsoft.com/en-us/azure/azure-percept/quickstart-percept-dk-set-up). Please make note of the following details as they will be required for deployment.
    * Region of IoT Hub
    * Resource group name which contains your Percept DK IoT Hub
    * IoT Hub Name
    * Edge device name

---

## Deploy the App through the Azure Portal

Deployment starts with this button. Please reference the below details for populating the requested deployment information.

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Funifiededgescenariostest.blob.core.windows.net%2Farm-template%2Fazure-percept%2Flatest%2FARM-template.json)

> Tip:  For a programmatic deployment alternative see [Programmatically Deploy the App with the Azure CLI](#programmatically-deploy-the-app-with-the-azure-cli).

The "Deploy to Azure" button will redirect you to the Azure portal with this deployment page:

![People Detector](docs/images/Custom-Deployment-Percept.png)
#

To deploy this reference solution, please enter the following parameters:

| Parameters | Description |
| ------ | ------ |
| __Region__ | Name of the region which host your existing Azure IoT Hub. |
| __Resource Group Device__ | Name of the resource group which host your Azure IoT Hub connected to Azure Percept DK. |
| __Resource Group AMS__ | Unique name of a new resource group to host Azure Media Service, Azure Data Lake and Front End application. |
| __Existing IoT Hub Name__ | Name of a the IoT Hub that your Azure Percept DK device is registered as an edge device. |
| __Existing Device Name__ | Name of the IoT Edge device in IoT Hub which is connected to Azure Percept DK. |
| __Service Principal Id__ | Id of an existing Service Principal which will be used in Azure Media Service. |
| __Service Principal Object Id__ | Object Id of an existing Service Principal which will be used in Azure Media Service. |
| __Service Principal Secret__ | Secret of an existing Service Principal which will be used in Azure Media Service. |
| __Password__ | A password to protect access to the web app which visualizes your output. A best practice is to assign a password to prevent others on the internet from seeing the testing output of your Percept DK. |


Once deployment is complete (in approximately 20-25min), you can launch the web application by navigating to the `Resource Group AMS` name selected above. You will see an Azure Web Services deployment which starts with `ues-perceptapp` followed by 4 random characters. Select this app, then chose the `Browse` button in the top left:

![Web Application](docs/images/Web-App-Launch.png)

Once the application loads, you will need to enter the password you entered at deployment time. The password is cached for subsequent visits to the same application.

> Tip: to successfully redeploy, the Role Assignment associated with the Service Principal will need to be deleted, if using the same Service Principal.  See [Remove Azure role assignments](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-remove) for a guide on how to do so.

## Programmatically Create a Service Principal with the Azure CLI

The process of programmatically creating a Service Principal through an App Registration using the Azure CLI is shown in this section.  This is an alternative to creating an App Registration and password/secret through the Azure Portal.  The Azure CLI must be installed on the machine from which the commands will be run.  If no Azure CLI is installed, follow [these steps](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) for the appropriate OS.

Log in interactively to Azure with the Azure CLI as follows:
```bash
az login
```

Choose the correct subscription (if there is more than one) as follows:
```bash
az account set --subscription <Azure subsciption name or ID>
```

To create an AAD Service Principal-based App Registration with a password/secret using the Azure CLI, perform the following (filling in the `<>` with new values - not including the `<>` symbols, however):

```bash
az ad sp create-for-rbac -n "<new name for app registration>"
```

The command above will provide the App ID and password.  To show more information such as the Object ID, use the following command:

```bash
az ad sp show --id "<app id>"
```

The command above should provide output with the App ID and Object ID along with the rest of the information regarding the App Registration and associated Service Principal.  The App ID and password/secret will be needed for the ARM template parameters in the deployment steps in the next section.

>  Tip:  Please check for pre-existing Role Assignments for the Azure Subscription (if the IoT Hub resource group has any from previous deployments then the deployment will not succeed).

To check for pre-existing Role Assignments use the following command:

```bash
az role assignment list --all
```

The `list` command will output potentially many responses similar to the one below:

```json
  {
    "canDelegate": null,
    "condition": null,
    "conditionVersion": null,
    "description": null,
    "id": "/subscriptions/<azure subscription id>/resourcegroups/<iot hub resource group>/providers/Microsoft.Authorization/roleAssignments/<assignment name>",
    "name": "<assignment name>",
    "principalId": "<service principal id>",
    "principalName": "",
    "principalType": "ServicePrincipal",
    "resourceGroup": "<iot hub resource group>",
    "roleDefinitionId": "/subscriptions/<azure subscription id>/providers/Microsoft.Authorization/roleDefinitions/<role definition id>",
    "roleDefinitionName": "Contributor",
    "scope": "/subscriptions/<azure subscription id>/resourcegroups/<iot hub resource group>",
    "type": "Microsoft.Authorization/roleAssignments"
  }
```

Take care to only delete the Role Assignments associated with the IoT Hub resource group with `Contributor` as the `roleDefinitionName` as these were the ones created for the Service Principals associated with previous deployments.  Delete Role Assignments associated with the IoT Hub resource group before redeploying (deleting just Managed Identities in the AMS resource group will _not_ delete the Role Assignments).  To delete a Role Assignment use the following command for each ID found (`id`, above):

```bash
az role assignment delete --ids <full id e.g. "/subscriptions/<azure subscription id>/resourcegroups/<iot hub resource group>/providers/Microsoft.Authorization/roleAssignments/<assignment name>">
```

The next section will walk through a programmatic deployment alternative to using the "Deploy to Azure" button.

## Programmatically Deploy the App with the Azure CLI

The process to programmatically deploy or redeploy the People Counting reference solution with the Azure CLI is shown in this section.  This is an alternative to using the "Deploy to Azure" button.  Ensure a Service Principal (App ID, Object ID and password/secret) is available from the AAD Application Registration process shown in the previous section or from [this how-to-guide](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal).  The Azure CLI must be installed on the machine from which the commands will be run.  If no Azure CLI is installed, follow [these steps](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) for the appropriate OS.

Create a parameters file that looks like this one shown to make the deployment from the command line easier (these are the user-defined parameters for the ARM template).  Fill in the `<>` parts in the template below and name this file `ARM-template.parameters.json` - placing it in the same directory as the main ARM template:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "resourceGroupDevice": {
            "value": "<name of existing resource group of the edge device and iot hub>"
        },
        "resourceGroupAMS": {
            "value": "<a new name for the media services resource group>"
        },
        "existingIotHubName": {
            "value": "<name of existing iot hub>"
        },
        "existingDeviceName": {
            "value": "<name of existing Percept edge device>"
        },
        "servicePrincipalId": {
            "value": "<service principal application id>"
        },
        "servicePrincipalSecret": {
            "value": "<service principal secret>"
        },
        "password": {
            "value": "<a new password for the web app>"
        }
    }
}
```

Log in interactively to the Azure CLI as follows if not logged in already:
```bash
az login
```

Choose the correct subscription (if there are more than one) as follows if this has not been done, yet:
```bash
az account set --subscription <Azure subsciption name or ID>
```

The following commands shows how to deploy with the Azure CLI into a subscription with a parameter template file (created in this section above):

> Tip: if the deployment fails, consider adding `--debug > deploy-log.txt 2>&1` to the end of the command below when running again for full, realtime logs, redirected to a file (they are long).  Ensure the clean-up steps below have been performed before redeploying.

```bash
az deployment sub create --location WestUS --name michhar-people-counting --template-file ARM-template.json --parameters @ARM-template.parameters.json
```

To clean up a deployment so that you may redeploy with the same parameters, perform the following with the Azure CLI:

> Tip:  to get all Role Assignments use `az role assignment list --all`

```bash
az deployment sub delete --name <name of deployment>
az group delete --name <name of AMS resource group>
az role assignment delete --ids <id of role assignment created in deployment>
```

# People Counting in a Zone

You can create a polygon region in the camera frame to count the number of people in the zone.  Metrics are displayed at the bottom showing total people in the frame vs. people in the zone.  To create a zone, click anywhere on the video window to establish the first corner of your polygon. Clicking 4 times will create a 4-sided polygon. People identified in the zone are shown with a yellow highlight.  Press the `Clear` button in the lower right to clear your zone definition.

## Additional App Details

Please see the [frontend-app documentation](frontend-app/app/README.md) for addition details on configuring and using the frontend application.

---
# Troubleshooting Common Deployment Errors
Please visit the [Troubleshooting Guide](docs/deployment-troubleshooting-guide.md) for debugging common deployment errors.
