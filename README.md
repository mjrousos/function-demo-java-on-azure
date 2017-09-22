# Walkthrough Steps

## Environment Setup

### Softwares

Install Azure functions core tools using

```batchfile
npm i -g azure-functions-core-tools@core
```

### Extensions

Navigate to [VSCode Java Debug extension Jenkins server](https://vscjavaci.cloudapp.net/), and use your GitHub account to log in. When everything works fine, download the [extension vsix file](https://vscjavaci.cloudapp.net/job/vscode-java-debug.vsix/19/Azure/).

> If the downloaded file is renamed to zip (for example when you download it from Microsoft Edge browser), you need to rename it back to `vscode-java-debug-0.1.0.vsix`

Launch Visual Studio Code, press `F1` and then type `>Extensions: Install from VSIX` in the command input box (the right-angle-bracket `>` is in the input box by default, so you don't need to duplicate it); then press `Enter`, and a dialog will be popped up, you need to choose the `vscode-java-debug-0.1.0.vsix` which is the one you just downloaded. After it is installed, Visual Studio Code will ask you to reload the whole window, and just do it.

Clone the [azure-functions-java-worker](https://github.com/Azure/azure-functions-java-worker) repository to your local folder using:

```batchfile
git clone https://github.com/Azure/azure-functions-java-worker.git
```

Enter the repository you just cloned by executing:

```batchfile
cd azure-functions-java-worker
```

And install it locally by running:

```batchfile
mvn clean install
```

Do similar things for other two packages: [azure-maven-archetypes](https://github.com/Microsoft/azure-maven-archetypes) and [azure-maven-plugins](https://github.com/Microsoft/azure-maven-plugins). You are required to clone all the packages in your working directory (which should just be the parent folder of `azure-functions-java-worker`).

```batchfile
git clone https://github.com/Microsoft/azure-maven-archetypes.git
cd azure-maven-archetypes
mvn clean install
cd ../
git clone https://github.com/Microsoft/azure-maven-plugins.git
cd azure-maven-plugins
mvn clean install
```

### Configurations

Add a new environment variable called `JAVA_OPTS`, and set its value to `-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005` (don't miss the leading dash there).

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

And make sure you replace the `$UniqueId$` in `walkthrough/pom.xml` with the actual string of your unique ID.

Then open `walkthrough/local.settings.json` and paste the **connection string** of your storage account to the value of `AzureWebJobsStorage`.

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

To terminate the app, press `Ctrl + C`, in addition, launch some Take Manager and **terminate all `java.exe` processes** (you don't need to make sure it disappears from the task manager because some of the `java.exe` will be automatically restarted; what you need to do is just trigger the `End Process` on every `java.exe`).

### Deployment

Try the following command to deploy the function app, and that's it.

```batchfile
mvn azure-functions:deploy
```

And you will find your functions app named `walkthrough-<UniqueID>` under `Java Demos` subscription.

You will also be able to find the URL of the deployed app within the last 10 lines of the command line output. So it is possible to verify that it is actually running on Azure.

```batchfile
curl -X POST -d "World" <Deployed Host URL>/api/hello
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
curl -X POST -d "World" <Deployed Host URL>/api/hello
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
