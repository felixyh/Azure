# Azure Python SDK Authentication

Written By Ronnie Quan

# Overview

Python is widely used in infrastructure as code type of project, I started my cloud journey utilizing Java and Python, then abandon them all to work in windows and Azure world.

Now finally I have chance to start from scratch again with Python.

In this article, I will install PyCharm, create a project from scratch, authenticate to Azure and work with keyvault.

The goal is to test from local without any credential stored in local, either in IDE or environment variable. We will utilize DefaultAzureCredential class instead of UsernamePasswordCredential.

When run the code from Azure, the best option is to use managed identity.

# Installation

> PyCharm

Download PyCharm Community Version(Free), and install in your local environment.

> Project and Virtual Environment

Create your first project and let the IDE to create Python virtual environment for you.

![img](https://miro.medium.com/max/1338/1*0uREnxiVQ9CGpCCQye5HWw.png)

> Azure Packages

```
pip install azure-keyvault-secrets azure-identity
```

# Coding

> Configure KeyVault Name

I put keyvault name in PyCharm run configurations

![img](https://miro.medium.com/max/1400/1*sdIAj6ondfwjThWXVJ_Qrg.png)

> Hello World Code

```
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient
import os

# Press the green button in the gutter to run the script.
if __name__ == '__main__':
    keyVaultName = os.environ["KEY_VAULT_NAME"]
    KVUri = f"https://{keyVaultName}.vault.azure.net"
    credential = DefaultAzureCredential(exclude_environment_credential=True, exclude_managed_identity_credential=True, exclude_visual_studio_code_credential=True, exclude_shared_token_cache_credential=True )
    secret_client = SecretClient(vault_url=KVUri, credential=credential)
    secret = secret_client.get_secret("test")
    print(secret.name)
    print(secret.value)
```

> Sign In to Azure

```
az login
```

1. If the CLI can open your default browser, it will do so and load an Azure sign-in page.
2. Otherwise, open a browser page at https://aka.ms/devicelogin and enter the authorization code displayed in your terminal.
3. Sign in with your account credentials in the browser.

https://docs.microsoft.com/en-us/azure/key-vault/secrets/quick-create-python#sign-in-to-azure

> Execute Code

From PyCharm, click run or debug button from top right, or use short cut key shift + F10, we retrieved our secret from Azure keyvault.

```
C:\Users\<user>\PycharmProjects\MyPythonProject\venv\Scripts\python.exe C:/Users/<user>/PycharmProjects/MyPythonProject/main.py
test
testProcess finished with exit code 0
```

> More About DefaultAzureCredential

In our code, we added multiple exclude arguments, why?

The default credential class get access token from following identity options, it evaluates the option one by one and stops when a token is provided.

1. A service principal configured by environment variables.
2. An Azure managed identity.
3. On Windows only: a user who has signed in with a Microsoft application, such as Visual Studio. If multiple identities are in the cache, then the value of the environment variable `AZURE_USERNAME` is used to select which identity to use. See SharedTokenCacheCredential for more details.
4. The user currently signed in to Visual Studio Code.
5. The identity currently logged in to the Azure CLI.
6. The identity currently logged in to Azure PowerShell.

In our case, we would like to use az cli logged in user to authenticate to Azure keyvault, since we don’t prefer service principal, we don’t have managed identity in local and we are not using Visual Studio, so we skip all the first four identity options.

```
credential = DefaultAzureCredential(exclude_environment_credential=True, exclude_managed_identity_credential=True, exclude_visual_studio_code_credential=True, exclude_shared_token_cache_credential=True )
```

By the way, when I don’t have exclude argument, the execution failed with exception in managed identity option. Not sure this is by design, but I just include exclude argument to get around.

```
Attempted credentials:
 EnvironmentCredential: EnvironmentCredential authentication unavailable. Environment variables are not fully configured.
 ManagedIdentityCredential: Unexpected content type "text/html; charset=utf-8"
Traceback (most recent call last):
  File "PycharmProjects/MyPythonProject/main.py", line 17, in <module>
```

# References

[Azure Key Vault Secret client library for Python | Microsoft Docs](https://docs.microsoft.com/en-us/python/api/overview/azure/keyvault-secrets-readme?view=azure-python)

[Quickstart - Azure Key Vault Python client library - manage secretsGet started with the Azure Key Vault secret client library for Python. Follow the steps below to install the package…docs.microsoft.com](https://docs.microsoft.com/en-us/azure/key-vault/secrets/quick-create-python#code-details)

[azure.identity.DefaultAzureCredential class | Microsoft Docs](https://docs.microsoft.com/en-us/python/api/azure-identity/azure.identity.defaultazurecredential?view=azure-python)

[azure.identity.UsernamePasswordCredential class | Microsoft Docs](https://docs.microsoft.com/en-us/python/api/azure-identity/azure.identity.usernamepasswordcredential?view=azure-python)

[Download PyCharm: Python IDE for Professional Developers by JetBrains](https://www.jetbrains.com/pycharm/download/#section=windows)