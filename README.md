# Walkthrough Steps

## One-Time Environment Setup

### Softwares & Extensions

Install Azure functions core tools using

```batchfile
npm i -g azure-functions-core-tools@core
```

Install [Java Extension Pack](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack) for Visual Studio Code.

### Configurations

Create a new storage account by executing (The `UniqueID` is just used to make sure your account name doesn't conflict with others, for example you can use the "current date & time", or "your alias + sequence number"; make sure the total length of the names are not exceeding 24 characters, and make sure you only use lower-case letters or numbers):

```batchfile
az group create -n BoothAzFunc<UniqueID> -l westus
az storage account create -n boothazfunc<UniqueID> -g BoothAzFunc<UniqueID> -l westus --sku Standard_LRS
```

Finally to get the storage **connection string** which will be used in the future:

```batchfile
az storage account show-connection-string -n boothazfunc<UniqueID> -g BoothAzFunc<UniqueID>
```

Copy it (Starting with `DefaultEndpointsProtocol...`) to somewhere.

### Source Code

Go to your working directory and clone the sample code repository from:

```batchfile
git clone https://github.com/Microsoft/function-demo-java-on-azure.git
cd function-demo-java-on-azure
```

And make sure you replace the [`$UniqueId$`](https://github.com/Microsoft/function-demo-java-on-azure/blob/master/walkthrough/pom.xml#L18) in `walkthrough/pom.xml` with the actual string of your unique ID. And you also need to replace [`$ResourceGroupName$`](https://github.com/Microsoft/function-demo-java-on-azure/blob/master/walkthrough/pom.xml#L82) in `walkthrough/pom.xml` with the actual string of your resource group name (which should be `BoothAzFunc<UniqueID>`).

Then open `walkthrough/local.settings.json` and paste the **connection string** of your storage account to the value of [`AzureWebJobsStorage`](https://github.com/Microsoft/function-demo-java-on-azure/blob/master/walkthrough/pom.xml#L82).

## DEMO 1: Java Functions by Maven

### Compiling and Running

Enter the folder `walkthrough`, and execute:

```batchfile
mvn clean package
```

Then you may run it by:

```batchfile
mvn azure-functions:run
```

The timer and the queue functions will be triggered once per 30 seconds. To trigger the HTTP function, you need to execute the following command in a new command line window:

```batchfile
curl -X POST -d "World" http://localhost:7071/api/hello
```

To terminate the app, press `Ctrl + C`.

### Deployment

Try the following command to deploy the function app, and that's it.

```batchfile
mvn azure-functions:deploy
```

And you will find your functions app named `walkthrough-<UniqueID>` under `Java Demos` subscription.

You will also be able to find the URL of the deployed app within the last several lines of the command line output. So it is possible to verify that it is actually running on Azure.

```batchfile
curl -X POST -d "Azure World" <Deployed Host URL>/api/hello
```

## DEMO 2: Java Functions by Visual Studio Code

Get to the built-in terminal by pressing the `Ctrl + Backtick`.

### Compiling and Running

Enter the folder `walkthrough` in the built-in terminal, and execute:

```batchfile
mvn clean package
```

Then you may run it by:

```batchfile
mvn azure-functions:run
```

The timer and the queue functions will be triggered once per 30 seconds. To trigger the HTTP function, you need to execute the following command in a new command line window:

```batchfile
curl -X POST -d "World" http://localhost:7071/api/hello
```

To terminate the app, press `Ctrl + C`, in addition, launch some Take Manager and **terminate all `java.exe` processes** (you don't need to make sure it disappears from the task manager because some of the `java.exe` will be automatically restarted; what you need to do is just trigger the `End Process` on every `java.exe`).

### Deployment

Try the following command to deploy the function app, and that's it.

```batchfile
mvn azure-functions:deploy
```

And you will find your functions app named `walkthrough-<UniqueID>` under `Java Demos` subscription.

You will also be able to find the URL of the deployed app within the last 10 lines of the command line output. So it is possible to verify that it is actually running on Azure.

```batchfile
curl -X POST -d "Azure World" <Deployed Host URL>/api/hello
```

## DEMO 3: Debugging Java Functions by Visual Studio Code

Get to the built-in terminal by pressing the `Ctrl + Backtick`.

### Debugging

Enter the folder `walkthrough` in the built-in terminal, and execute:

```batchfile
mvn clean package
```

Navigate to the Debug page of Visual Studio Code by pressing `Ctrl + Shift + D`; then click the little gear icon (Open launch.json) besides "No Configurations" dropdown; and select "Java".

In the newly created `launch.json`, locate the `Debug (Attach)` block, and set the `port` to be `5005`.

Now you run the functions app by:

```batchfile
mvn azure-functions:run
```

Wait for the functions host to start, and select the `Debug (Attach)` item in the dropdown list, and click the little green run icon (Start debugging) beside that dropdown list.

Now you can set the breakpoints in:

* [Line 13 of `Http.java`](https://github.com/Microsoft/function-demo-java-on-azure/blob/master/walkthrough/src/main/java/com/microsoft/azure/functions/Http.java#L13)
* [Line 9 of `Queue.java`](https://github.com/Microsoft/function-demo-java-on-azure/blob/master/walkthrough/src/main/java/com/microsoft/azure/functions/Queue.java#L9)
* [Line 10 of `Timer.java`](https://github.com/Microsoft/function-demo-java-on-azure/blob/master/walkthrough/src/main/java/com/microsoft/azure/functions/Timer.java#L10)

Those breakpoints will be hit when the corresponding function is triggered. And when any of them is hit, hover the mouse to some variables, like `executionContext` or `timerInfo`, to demostrate that the user could inspect all the execution environment here. And press `F5` to continue running.

## DEMO 4: Deploy Java Functions by Jenkins

### Prerequisites

1. Install the [Azure Function Plugin](https://github.com/jenkinsci/azure-function-plugin) for Jenkins. You can install/update the plugin in Jenkins Update Center (Manage Jenkins -> Manage Plugins, search Azure Function Plugin).
2. Create an Azure Service Principal through [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli) or [Azure Portal](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal).
3. Open Jenkins dashboard, go to Credentials, add a new Microsoft Azure Service Principal with the credential information you just created.

### Deploy using Freestyle Project

You can deploy your Azure Function using a classic freestyle project:

1. Create a new freestyle project in Jenkins, add necessary build steps to build your code. For example, if you use maven commands above you should get following directory structure after build:
```
target/azure-functions
-- walkthrough-prototype
    |-- Queue
        |-- function.json
    |-- Timer
        |-- function.json
    |-- hello
        |-- function.json
    |-- host.json
    |-- walkthrough-1.0-SNAPSHOT.jar
```
2. Add a post-build action 'Publish an Azure Function'.
3. Select your Azure credential in Azure Profile Configuration section.
4. In App Configuration section, choose the resource group and function app in your subscription.
5. Fill in 'Files' field with `**/*.jar,**/*.json` to select files we want to deploy.
6. Fill in 'Source Directory' field with `target/azure-functions`.
6. Save the project and build it, your function app will be deployed to Azure when build is completed.

### Deploy using Pipeline

You can also use this plugin in pipeline (Jenkinsfile). Here are some samples to use the plugin in pipeline script:

To deploy a Java function app:

```groovy
azureFunctionAppPublish azureCredentialsId: '<credential_id>',
    resourceGroup: '<resource_group_name>', appName: '<app_name>',
    filePath: '**/*.jar,**/*.json', sourceDirectory: 'target/azure-functions'
```
For advanced options, you can use Jenkins Pipeline Syntax tool to generate a sample script.
