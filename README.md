# Walkthrough Steps

## Environment Setup

### Softwares

Install Azure functions core tools using

```
npm i -g azure-functions-core-tools@core
```

### Extensions

Navigate to [VSCode Java Debug extension Jenkins server](https://vscjavaci.cloudapp.net/), and use your GitHub account to log in. When everything works fine, download the [extension vsix file](https://vscjavaci.cloudapp.net/job/vscode-java-debug.vsix/19/Azure/).

> If the downloaded file is renamed to zip (for example when you download it from Microsoft Edge browser), you need to rename it back to `vscode-java-debug-0.1.0.vsix`

Launch Visual Studio Code, press `F1` and then type `>Extensions: Install from VSIX` in the command input box (the right-angle-bracket `>` is in the input box by default, so you don't need to duplicate it); then press `Enter`, and a dialog will be popped up, you need to choose the `vscode-java-debug-0.1.0.vsix` which is the one you just downloaded. After it is installed, Visual Studio Code will ask you to reload the whole window, and just do it.

Clone the [azure-functions-java-worker](https://github.com/Azure/azure-functions-java-worker) repository to your local folder using:

```
git clone https://github.com/Azure/azure-functions-java-worker.git
```

Enter the repository you just cloned by executing:

```
cd azure-functions-java-worker
```

And install it locally by running:

```
mvn clean install
```

Do similar things for other two packages: [azure-maven-archetypes](https://github.com/Microsoft/azure-maven-archetypes) and [azure-maven-plugins](https://github.com/Microsoft/azure-maven-plugins). You are required to clone all the packages in your working directory (which should just be the parent folder of `azure-functions-java-worker`).

```
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

```
az group create -n BoothAzFunc<UniqueID> -l westus
az storage account create -n boothazfunc<UniqueID> -g BoothAzFunc<UniqueID> -l westus --sku Standard_LRS
```

Finally to get the storage **connection string** which will be used in the future:

```
az storage account show-connection-string -n boothazfunc<UniqueID> -g BoothAzFunc<UniqueID>
```

Copy it (Starting with `DefaultEndpointsProtocol...`) to somewhere.

## DEMO 1: Java Functions by Maven

### Scaffolding

Launch a command line window (**Don't** use `PowerShell` if you are in windows, use `cmd.exe` instead), and enter your working directory.

Execute the following command to create a new project:

```
mvn archetype:generate -DarchetypeGroupId=com.microsoft.azure -DarchetypeArtifactId=azure-functions-archetype -DarchetypeVersion=1.0-SNAPSHOT
```

It is an interactive command, so you also need to enter the following values where the command line tool prompts:

```
Define value for property 'groupId': : com.microsoft.azure.boothdemo
Define value for property 'artifactId': : walkthrough
Define value for property 'version':  1.0-SNAPSHOT: : <Enter>
Define value for property 'package':  com.microsoft.azure.boothdemo: : <Enter>
Define value for property 'appName':  ******: : walkthrough-<UniqueID>
Confirm properties configuration:  ****** Y : : <Enter>
```

Enter the project folder by:

```
cd walkthrough
```

### Coding

Open `pom.xml` using your favorite text editor and search for `azure-functions-java-core`. You should now be located at some where within a `<dependency>` block. Replace the whole `<dependency>` block with:

```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure-functions-java-core</artifactId>
    <version>1.1-SNAPSHOT</version>
</dependency>
```

Rename `src/main/java/com/microsoft/azure/boothdemo/Function.java` to `src/main/java/com/microsoft/azure/boothdemo/Http.java`. Then open that file and rename the class named `Function` to `Http`:

```java
public class Http {
    ...
}
```

Actually you will also need to rename a whole bunch of things, but to make the things easier, just delete the entire `src/test` folder.

Now go back to the root directory of walkthrough, and add a timer function by:

```
mvn azure-functions:add
```

This is an interactive command as well:

```
<Select 6. TimerTrigger>
<Function Name: timer>
<Package Name: com.microsoft.azure.boothdemo>
<schedule: */30 * * * * *>
```

Similarly, add a queue function by:

```
mvn azure-functions:add
```

And type in the values:

```
<Select 2. QueueTrigger>
<Function Name: queue>
<Package Name: com.microsoft.azure.boothdemo>
<connection: AzureWebJobsStorage>
<queueName: timerqueue>
```

Open `local.settings.json` and paste the **connection string** of your storage account to the value of `AzureWebJobsStorage`.

Open `src/main/java/com/microsoft/azure/boothdemo/Queue.java` and replace Line 10 with:

```java
executionContext.getLogger().info("Queue trigger input: " + myQueueItem);
```

Open `src/main/java/com/microsoft/azure/boothdemo/Timer.java` and replace Line 7 ~ 10 with:

```java
@FunctionName("Timer")
@QueueOutput(name = "myQueueItem", queueName = "timerqueue", connection = "AzureWebJobsStorage")
public String functionHandler(@TimerTrigger(name = "timerInfo", schedule = "*/30 * * * * *") String timerInfo, final ExecutionContext executionContext) {
    executionContext.getLogger().info("Timer trigger input: " + timerInfo);
    return "From timer: \"" + timerInfo + "\"";
}
```

### Compiling and Running

Enter the root folder of `walkthrough`, and execute:

```
mvn clean package
```

Then you may run it by (maybe not work for now, because we are waiting for the new deployment of Azure Functions team):

```
mvn azure-functions:run
```

The timer and the queue functions will be triggered once per 30 seconds. To trigger the HTTP function, you need to execute the following command in a new command line window:

```
curl -X POST -d "World" http://localhost:7071/api/hello
```

To terminate the app, press `Ctrl + C`, in addition, launch some Take Manager and **terminate all `java.exe` processes** (you don't need to make sure it disappears from the task manager because some of the `java.exe` will be automatically restarted; what you need to do is just trigger the `End Process` on every `java.exe`).

### Deployment

Try the following command to deploy the function app, and that's it.

```
mvn azure-functions:deploy
```

And you will find your functions app named `walkthrough-<UniqueID>` under `Java Demos` subscription.

## DEMO 2: Java Functions by Visual Studio Code

### Scaffolding

Launch a command line window (**Don't** use `PowerShell` if you are in windows, use `cmd.exe` instead), and enter your working directory.

Execute the following command to create a new project:

```
mvn archetype:generate -DarchetypeGroupId=com.microsoft.azure -DarchetypeArtifactId=azure-functions-archetype -DarchetypeVersion=1.0-SNAPSHOT
```

It is an interactive command, so you also need to enter the following values where the command line tool prompts:

```
Define value for property 'groupId': : com.microsoft.azure.boothdemo
Define value for property 'artifactId': : walkthrough
Define value for property 'version':  1.0-SNAPSHOT: : <Enter>
Define value for property 'package':  com.microsoft.azure.boothdemo: : <Enter>
Define value for property 'appName':  ******: : walkthrough-<UniqueID>
Confirm properties configuration:  ****** Y : : <Enter>
```

Launch Visual Studio Code by

```
code walkthrough
```

### Coding & Running

Then press `F1`, and type in `>Terminal: Select Default Shell` (the right-angle-bracket `>` is in the input box by default, so you don't need to duplicate it), and select `Command Prompt` (or `bash` on non-Windows machines). Then get to the built-in terminal by pressing the `Ctrl + Backtick`.

And finish all the `Coding` & `Compiling and Running` sections in `DEMO 1: Java Functions by Maven` within Visual Studio Code (editing files using Code editor, and executing commands in the built-in terminal).

### Deployment

Try the following command to deploy the function app, and that's it.

```
mvn azure-functions:deploy
```

And you will find your functions app named `walkthrough-<UniqueID>` under `Java Demos` subscription.

## DEMO 3: Debugging Java Functions by Visual Studio Code

### Coding

Make sure you have finished `Scaffolding` & `Coding & Running` sections in `DEMO 2: Java Functions by Visual Studio Code`.

### Debugging

First terminate all running instances.

Navigate to the Debug page of Visual Studio Code by pressing `Ctrl + Shift + D`; then click the little gear icon (Open launch.json) besides "No Configurations" dropdown; and select "Java".

In the newly created `launch.json`, locate the `Debug (Attach)` block, and set the `port` to be `5005`.

Now you run the functions app by:

```
mvn azure-functions:run
```

Wait for the functions host to start, and select the `Debug (Attach)` item in the dropdown list, and click the little green run icon (Start debugging) beside that dropdown list.

Now you can set the breakpoints in:

* Line 13 of `src/main/java/com/microsoft/azure/boothdemo/Http.java`
* Line 9 of `src/main/java/com/microsoft/azure/boothdemo/Queue.java`
* Line 10 of `src/main/java/com/microsoft/azure/boothdemo/Timer.java`

Those breakpoints will be hit when the corresponding function is triggered. And when any of them is hit, hover the mouse to some variables, like `executionContext` or `timerInfo`, to demostrate that the user could inspect all the execution environment here. And press `F5` to continue running.

## DEMO 4: Deploy Java Functions by Jenkins
