# Instructions to Add a "Deploy to Azure" Button to Your Markdown File

*Shane Neff*

*Senior Cloud Solution Architect- Microsoft*

*5/30/2022*

---

# Table of contents

- [Instructions to add a "deploy to Azure" button to your markdown file](#instructions-to-add-a-deploy-to-azure-button-to-your-markdown-file)
  - [Microsoft documentation](#microsoft-documentation)
  - [Steps](#steps)

---

## Microsoft documentation

Microsoft Docs: https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/deploy-to-azure-button

## Steps

1. Find URL to template file
2. Select "Raw" for unformatted text
3. Grab URL from top of template
4. Take the URL copied and paste it in the string below (example, replace what's there now)

```powershell
$url = "https://raw.githubusercontent.com/nanigan/templates/main/FunctionApp/Function.bicep"
[uri]::EscapeDataString($url)
```

5. Sample of the output of the Powershell:

```powershell
https%3A%2F%2Fraw.githubusercontent.com%2Fnanigan%2Ftemplates%2Fmain%2FFunctionApp%2FFunction.bicep
```

6. Copy the output of the command
7. Paste copied text after "uri/"

```powershell
(https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fnanigan%2Ftemplates%2Fmain%2FKeyVault%2FKeyVault.bicep)
```

8. Copy the "deploy to Azure" URL:

```json
[![Deploy to Azure](https://aka.ms/deploytoazurebutton)]
```

9. Combine link to button with URI and paste into readme file:

```json
[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fnanigan%2Ftemplates%2Fmain%2FKeyVault%2FKeyVault.bicep)
```

10. Final result

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fnanigan%2Ftemplates%2Fmain%2FKeyVault%2FKeyVault.bicep)



