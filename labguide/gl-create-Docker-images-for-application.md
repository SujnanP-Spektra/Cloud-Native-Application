# Exercise 1: Build Docker Images for the Application

### Estimated Duration: 60 minutes

## Lab scenario

In this exercise, you will learn how to containerize the Contoso Traders application using Docker images. Containerized applications are applications that run in isolated runtime environments called containers. A Docker image is a file used to execute code in a Docker container. Docker images act as a set of instructions to build a Docker container, like a template. Also, you will be pushing the created Docker images to the Azure Container Registry.

## Lab objectives

In this lab, you will complete the following tasks:

- Task 1: Set up a local infrastructure with the Linux VM
- Task 2: Build Docker images to containerize the application and push them to the container registry

### Task 1: Set up a local infrastructure with the Linux VM

In this task, you will be connecting to the Build agent VM using the Command Prompt and will be cloning the Contoso trader website GitHub repo.  

1. Once you log into the VM, search for **cmd** **(1)** in the Windows search bar and click on **Command Prompt** **(2)** to open.

   ![](media/latest-ex1-opencmd.png "open cmd")
    
1. Run the given command **<inject key="Command to Connect to Build Agent VM" enableCopy="true" />** to connect to the Linux VM using ssh.
   
   >**Note**: In the command prompt, type **yes** and press **Enter** for `Are you sure you want to continue connecting (yes/no/[fingerprint])?`
   
1. Once the SSH is connected to the VM, please enter the VM password given below:
   
    * Password: **<inject key="Build Agent VM Password" enableCopy="true" />**

        ![](media/E1T1S3.png "open cmd")
   
        >**Note**: Please note that while typing the password, you won’t be able to see it due to security concerns.
    
1. Once the VM is connected, run the following command to clone the GitHub repository that we are going to use for the lab.

    ``` 
    git clone https://github.com/CloudLabsAI-Azure/Cloud-Native-Application
    ```
    
    ![](media/cdn-nat-lab3-e31-g2.png)
    
    > **Note:** If you receive an output message stating that the destination path 'Cloud-Native-Application' already exists and is not an empty directory. Please run the following commands and then reperform step 4 of the task.

     ```
    sudo su
    rm -rf Cloud-Native-Application
    exit
     ```   
     ![](media/cdn-nat-lab3-e31-g1.png)
    
1. After the GitHub cloning is completed, run the command below to change the directory to the lab files.
    
    ```
    cd Cloud-Native-Application/labfiles/ 
    ```
    
    ![](media/cdn-nat-lab3-e31-g3.png)
    
### Task 2: Build Docker images to containerize the application and push them to the container registry

In this task, you will be building the Docker images to containerize the application and will be pushing them to the ACR (Azure Container Registry) to later use in AKS.

1. Run the below command to log in to Azure, navigate to the device login URL `https://microsoft.com/devicelogin` in the browser, and copy the authentication code.

   ``` 
   az login
   ```
    
   ![](media/EX1-T2-S1.png)

   > **Note:** If `az login` isn’t recognized, install Azure CLI using the commands below, let the setup complete, and then rerun `az login` to proceed

   ``` 
   sudo apt update
   sudo apt install azure-cli -y
   ```
    
1. Enter the copied authentication code **(1)** and click on **Next** **(2)**.

   ![](media/cdn-nat-lab3-e31-g4.png)
   
1. On the **Sign in to your account** tab you will see a login screen, in that enter the following email/username and then click on **Next**.

   * Email: **<inject key="AzureAdUserEmail"></inject>**

        ![](media/cdn-nat-lab3-e31-g5.png)

1. Now enter the following Temporary Access Pass and click on **Sign in**.

   * Temporary Access Pass: **<inject key="AzureAdUserPassword"></inject>**

        ![](media/cdn-nat-lab3-e31-g6.png)

        > **Note:** You will not get the pop-up to enter the password if you had gotten the **Pick an account** pop-up where you had chosen the account.

1. In a pop-up to confirm the sign-in to Microsoft Azure CLI, click on **Continue**.

   ![](media/cdn-nat-lab3-e31-g7.png)

1. After signing in, you will see a confirmation pop-up: **You have signed in to the Microsoft Azure Cross-platform Command Line Interface application on your device**. Close the browser tab and open the previous Command Prompt session.   

   ![](media/ex1-t2-step6-signin-confirm.png)

1. Once you log in to Azure, you are going to build the Docker images in the next steps and will be pushing them to ACR.

   ![](media/cdn-nat-lab3-e31-g8.png)

1. Please make sure that you are in the **labfiles** directory before running the next steps, as the Docker build needs to find the DockerFile to create the image.

    ```
    cd Cloud-Native-Application/labfiles/
    ```
    
1. Now build the **contosotraders-carts** Docker image using the Dockerfile in the directory. Take note of how the deployed Azure Container Registry is referenced.

    ```
    docker build src -f ./src/ContosoTraders.Api.Carts/Dockerfile -t contosotradersacr<inject key="DeploymentID" enableCopy="true"/>.azurecr.io/contosotradersapicarts:latest -t contosotradersacr<inject key="DeploymentID" enableCopy="true"/>.azurecr.io/contosotradersapicarts:latest
    ```
    
    ![](media/cdn-nat-lab3-e31-g9.png)
    
1. Repeat the steps to create the **contosotraders-Products** Docker image with the below command.

    ```
    docker build src -f ./src/ContosoTraders.Api.Products/Dockerfile -t contosotradersacr<inject key="DeploymentID" enableCopy="true"/>.azurecr.io/contosotradersapiproducts:latest -t contosotradersacr<inject key="DeploymentID" enableCopy="true"/>.azurecr.io/contosotradersapiproducts:latest
    ```

    ![](media/cdn-nat-lab3-e31-g10.png)

1. Run the below command to change the directory to `services` and open the `configService.js` file.

    ```
    cd src/ContosoTraders.Ui.Website/src/services
    sudo chmod 777 configService.js
    vi configService.js
    ```
    
    ![](media/cdn-nat-lab3-e31-g11.png)
    
1. In the `vi` editor, press **_i_** to get into the `insert` mode. In the APIUrl and APIUrlShoppingCart, replace **deploymentid** with **<inject key="DeploymentID" enableCopy="true"/>** value and **REGION** with **<inject key="Region" enableCopy="true"/>** value. Then press **_ESC_**, write **_:wq_** to save your changes, and close the file. We need to update the API URL here so that the Contoso Traders application can connect to the product API once it's pushed to AKS containers.
    
    >**Note**: If **_ESC_** doesn't work press `ctrl + [` and then write **_:wq_** to save you changes and close the file.         
    Still, if there are any issues while saving the file, connect to the LabVM using RDP with the credentials provided in the Environment details tab and perform the above step.
    
    ```
    const APIUrl = 'http://contoso-traders-productsdeploymentid.REGION.cloudapp.azure.com/v1';
    const APIUrlShoppingCart = 'https://contoso-traders-cartsdeploymentid.orangeflower-95b09b9d.REGION.azurecontainerapps.io/v1';
    ```

    ![](media/cdn-nat-lab3-e31-g12.png)

    ![](media/cdn-nat-lab3-e31-g14.png)

1. Run the below command to change the directory to the `ContosoTraders.Ui.Website` folder.

    ```
    cd
    cd Cloud-Native-Application/labfiles/src/ContosoTraders.Ui.Website
    ```

1. Now build the **contosotraders-UI-Website** docker image with the below command. Make sure to replace the SUFFIX with the given DeploymentID **<inject key="DeploymentID" enableCopy="true"/>** value in the below command.

    ```
    docker build . -t contosotradersacr[SUFFIX].azurecr.io/contosotradersuiweb:latest -t contosotradersacr[SUFFIX].azurecr.io/contosotradersuiweb:latest
    ```    
    
    ![](media/cdn-nat-lab3-e31-g13.png)
    
    >**Note**: Please be aware that the above command may take up to 5 minutes to finish the build. Before taking any further action, make sure it runs successfully. Also, you may notice a few warnings related to npm version update, which is expected and doesn't affect the lab's functionality.
    
1. Redirect to the **labfiles** directory before running the next steps.

    ```
    cd
    cd Cloud-Native-Application/labfiles/
    ```

1. Observe the built Docker images by running the command `docker image ls`. The images are tagged with the latest; it is also possible to use other tag values for versioning.

    ```
    docker image ls
    ```

    ![](media/cdn-nat-lab3-e31-g16.png)

1. In the **Navigate** section, select **Resource groups**.

    ![](media/cdn-nat-lab3-e31-g17.png)

1. In the **Resource groups** list, select **ContosoTraders-<inject key="DeploymentID" enableCopy="false" />**.

    ![](media/cdn-nat-lab3-e31-g18.png)

1. In the **Resources** list, search for **contosotradersacr<inject key="DeploymentID" enableCopy="false" /> (1)** and select it (2).

    ![](media/cdn-nat-lab3-e31-g19.png)
   
1. In **contosotradersacr<inject key="DeploymentID" enableCopy="false" />**, expand **Settings (1)**, select **Access keys (2)**, and copy the **Password (3)**.

    ![](media/cdn-nat-lab3-e31-g20.png)  

1. Now switch back to **Command Prompt** and log in to ACR using the below command. Please update the Suffix and ACR password value in the command below. You should be able to see the output below in the screenshot. Make sure to replace the SUFFIX with the given DeploymentID **<inject key="DeploymentID" enableCopy="true"/>** value and password with the copied container registry password, which you have copied in the previous step, in the below command.

    ```
    docker login contosotradersacr[SUFFIX].azurecr.io -u contosotradersacr[SUFFIX] -p [password]
    ```

    ![](media/cdn-nat-lab3-e31-g21.png)

1. Once you log in to the ACR, please run the commands below to push the Docker images to the Azure container registry. Also, ensure that you update the SUFFIX value with the given DeploymentID **<inject key="DeploymentID" enableCopy="true"/>** value in the below commands.

   ```
   docker push contosotradersacr[SUFFIX].azurecr.io/contosotradersapicarts:latest 
   ```
   
   ```
   docker push contosotradersacr[SUFFIX].azurecr.io/contosotradersapiproducts:latest
   ```
   
   ```
   docker push contosotradersacr[SUFFIX].azurecr.io/contosotradersuiweb:latest
   ```
   
1. You should be able to see the Docker image getting pushed to ACR as shown in the screenshot below. 
    
    ![](media/cdn-nat-lab3-e31-g23.png)

## Summary

In this exercise, you have completely containerized your web application with the help of docker and pushed it to the container registry.
