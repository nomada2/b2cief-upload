# PowerShell to configure and upload a set of B2C IEF policies

## Purpose
PowerShell script which:
1. Modifies the xml of a set of IEF policies replacing them with values from the target B2C tenant and an optional configuration (useful if policies need to be used in different tenants - Dev, QA, etc. - with different REST urls, key names, etc.) 
2. Optionally uploads the files to a B2C tenants

## Usage

**Please install [AzureADPreview module](https://docs.microsoft.com/en-us/powershell/azure/active-directory/install-adv2?view=azureadps-2.0#installing-the-azure-ad-module) before proceding. You must also uninstall
any non-preview versions of the AzureAD module (uninstall-module AzureAD).**

### Tenant setup

If you have never set up your B2C to use IEF policies you can use [my IEF setup website](https://b2ciefsetup.azurewebsites.net/) or follow [instructions provided in the official documentation](https://docs.microsoft.com/en-us/azure/active-directory-b2c/custom-policy-get-started) to do so. 

### Policy setup
1. Download the script file and execute it in a PowerSehll console to define the two functions included in it (there may be better way of doing this but I am not yet that good at PowerShell).
1. Store your policies in a single folder. (The SampleData folder on this github project was downloaded from the [starter pack](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack) for local acounts).
2. Optionally, modify the sampleData/appSettings.json file to include any values you need to replace in the policies (e.g. REST API urls). The IEF resource and proxy app ids will be retrieved automatically from your B2C tenant - no need to provide them in the settings file.

The script will use the following string replacement rules to apply your *appSettings.json* values.

| appSettings property | String replaced in policy |
| -------- | ------ |
| PolicyPrefix | Inserted into the name of policies, e.g. *B2C_1A_MyTrustBase* where *My* is the value of the PolicyPrefix. Makes it easier to handle several sets of IEF policies in the tenant |
| ProxyIdentityExperienceFrameworkAppId | See [IEF applications setup](https://docs.microsoft.com/en-us/azure/active-directory-b2c/active-directory-b2c-get-started-custom?tabs=applications#register-identity-experience-framework-applications) |
| IdentityExperienceFrameworkAppId | See [IEF applications setup](https://docs.microsoft.com/en-us/azure/active-directory-b2c/active-directory-b2c-get-started-custom?tabs=applications#register-identity-experience-framework-applications) |
| *other* | You can define your own symbolic properties, e.g. *"CheckPlayerAPIUrl": "https://myapi.com"*. If you do, modify the PowerShell script to use the value of the property as replacement in policies with an appropriate rule to select which text should be replacedg. Look for *{CheckPlayerAPIUrl}* string in both the *TrustFrameworkExtensions.xml* and the *Upload-IEFPolicies.ps1* script to see an example |

### Upload-IEFPolicies

Use *Upload-IEFPolicies* function to upload your IEF policies to the B2C tenant you are currently signed into.

E.g.

```PowerShell
Connect-AzureAD -TenantId yourtenant.onmicrosoft.com
$confFile = 'C:\LocalAccounts\appSettings.json'
$source = 'C:\LocalAccounts'
$dest = 'C:\LocalAccounts\updated'

Upload-IEFPolicies  -sourceDirectory $source -configurationFilePath $confFile -updatedSourceDirectory $dest`
```
Parameters:

| Property name | Required | Purpose |
| -------- | ------ | ----- |
| sourceDirectory | Y | Directory path where your xml policies are stored |
| updatedSourceDirectory | N | Directory path where the policies updated by this script will be stored. Also used to prevent uploading unmodified policies |
| configurationFilePath | N | jso file with additional replacement strings. The script will match any property in this file with a string with format *{<property name>}* and replace it with the value of the property |
| generateOnly | N | If used, the script will only generate policy files but not upload them to B2C |
| prefix | N | String inserted into the name of generated policies, e.g. the new base policy name will be *B2C_1A_XYZTrustFrameBase, where XYZ is the value of the provided prefix |








