# Migrate to ATP

## Introduction

The final lab of this workshop will guide you on how to set up a simple migration to ATP using Goldengate Microservices. By using Oracle GoldenGate Microservices on Oracle Cloud Marketplace, replication from on-premises to cloud and cloud-to-cloud platforms can easily be established and managed. 
This will allow you to deploy Oracle GoldenGate in an off-box architecture, which means you can run and manage your Oracle GoldenGate deployment from a single location.

*Estimated lab time*: 30 minutes

### Objectives

In the final lab of the workshop, we will configure the replication process in Microservices and apply captured changes from the source database to our target Autonomous Database.

For a technical overview of this lab step, please watch the following video:

[](youtube:x7vX5r3Qzns)

### Prerequisites

* This lab assumes that you completed all preceding labs, and ready to migrate to ATP.

## **Step 1**: Access to Goldengate Microservices Instance

1. After successful creating three extract processes, now it is time to explore your GG Microservices server. Let's make a console connection to Microservice. Copy the IP address of `OGG_Microservices_Public_ip` from your note and connect using:

	**`ssh opc@your_microservice_ip_address -i ~/.ssh/oci`**

## **Step 2**: Retrieve Admin Password

1. Administrator user credentials of GoldenGate Microservices Web console is already created for you. Once you are in the instance, issue following and copy a credential value from the output.

	```
	<copy>
	cat ogg-credentials.json
	</copy>
	```

	![](/images/oggadmin.png)

2. Good practice is to keep it in your notepad. You will use it very often in the next steps.

## **Step 3**: Login to Microservices Web Console

1. Open your web browser and point to `https://your_microservices_ip_address`. Provide oggadmin in username and password which you copied, then log in.

	![](/images/gg_oggadmin.png)

## **Step 4**: Open Target Receiver Server

1. Then click on Target Receiver server's port **9023**, it will redirect you to a new tab. Provide your credentials again for username **oggadmin**.

	![](/images/gg_oggadmin_0.png)

2. You should be seeing something like the below image. This means that your extdmp is pumping captured trail files to your Microservices.

	![](/images/gg_oggadmin_1.png)

	This is something you will need if you want continuous replication and migration. 

## **Step 5**: Open Target Administration Server

1. In this lab scope, we will only migrate to ATP with help of initload. Click on Target Receiver server port **9021**, it will redirect you to new tab. Provide your credentials again for username **oggadmin**.

	![](/images/micro_oggadmin_0.png)

## **Step 6**: Modify Goldengate Credentials

1. You should be seeing the empty Extracts and Replicats dashboard. Let's add Autonomous Database credentials. Open the hamburger menu on the top-left corner, choose **Configuration**

	![](/images/micro_ggadmin_0.png)

2. It will open OGGADMIN Security and you will see we already have a connection to **HOL Target ATP** database. However, you still need to add a password here. Click on a pencil icon to **alter credentials**.

	![](/images/micro_ggadmin_1.png)

## **Step 7**: Update Password and Check Connection

1. Provide the password `GG##lab12345` and verify it. This is your ggadmin password, which we provided in lab 3.

	![](/images/micro_ggadmin_2.png)

2. After that click on **Log in** database icon.

	![](/images/micro_ggadmin_3.png)

## **Step 8**: Add Checkpoint Table

3. Scroll down to the **Checkpoint** and click on **+** icon, then provide `ggadmin.chkpt` and **SUBMIT**. 

	![](/images/micro_ggadmin_4.png)

	The checkpoint table contains the data necessary for tracking the progress of the Replicat as it applies transactions to the target system. Regardless of the Replicat that is being used, it is best practice to enable the checkpoint table for the target system.

4. Now let's go back to **Overview** page from here.

	![](/images/micro_ggadmin_5.png)

## **Step 9**: Add Replication Process

1. The apply process for replication, also known as Replicat, is very easy and simple to configure. There are four types of Replicats supported by the Oracle GoldenGate Microservices. On the overview page, go to Replicat part and click on **+** to create our replicat process.

	![](/images/micro_initload_0.png)

2. We will choose **Non-Integrated Replicat** for initial load, click **Next**. In non-integrated mode, the Replicat process uses standard SQL to apply data directly to the target tables. In our case, the number of records in the source database is small and we don't need to run in parallel, therefore it will suffice.

	![](/images/micro_initload_1.png)

## **Step 10**: Modify Replication Parameters

1. Provide your name for the replicat process, for example, **initload**, the process name has to be unique and 8 characters long. It is better if you give some meaningful names to identify them later on. Let's name it as **initload**, because this is currently our initial load process.

	![](/images/micro_initload_2_1.png)

2. Then click on the **Credentials Domain** drop-down list. There is only one credential at the moment, choose the available option for you. In the **Credential Alias**, choose **hol_tp** from the drop-down, which is the pre-created connection group to target ATP. 

	![](/images/micro_initload_2_2.png)

3. Scroll below and find "Trail Name", add **il** as trail name, because we defined this in our extract parameter, so it _**cannot**_ be a random name.

	![](/images/micro_initload_2_3.png)

4. Also provide **/u02/trails** in the "Trail Subdirectory" and choose a **Checkpoint Table** from the drop-down list. It is **GGADMIN.CHKPT** in our case.

5. Review everything then click **Next**

## **Step 11**: Edit Parameter File

1. Microservices has created a draft parameter file for your convenience. Erase only below line from the existing draft parameter:

	```MAP *.*, TARGET *.*```

	![](/images/micro_initload_3_1.png)

2. Then add below configuration instead:

	```
	<copy>
	MAP public."Countries", TARGET Parking.Countries;
	MAP public."Cities", TARGET Parking.Cities;
	MAP public."Parkings", TARGET Parking.Parkings;
	MAP public."ParkingData", TARGET Parking.ParkingData;
	MAP public."PaymentData", TARGET Parking.PaymentData;
	</copy>
	```
	
	![](/images/micro_initload_3_2.png)

3. Make sure everything is correct until this stage. Click **Create and Run** to start our replicat.

	![](/images/micro_initload_4.png)

## **Step 12**: Check Replication Status

1. In the overview dashboard, you should now be seeing the running INITLOAD replication. Click on **Action** button, choose **Details**.

	![](/images/micro_initload.png)
	
2. You can see the details of the running replicat process. In the statistics tab, you can see some changes right away. 

	![](/images/micro_initload_5.png)

Congratulations! You have completed this workshop! 

You successfully migrated PostgreSQL database to Autonomous Database in Oracle Cloud Infrastructure.

## Summary

Here is a summary of resources which was created by Terraform script and used in our workshop.

1. [Virtual Cloud Network](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingVCNs.htm)
- Public Subnet, Internet Gateway
- Private Subnet, NAT Gateway, Service gateway

2. [Compute Virtual Machines and Shapes, OS Images](https://docs.oracle.com/en-us/iaas/Content/Compute/References/computeshapes.htm)
- Source PostgreSQL database instance, 
- Goldengate PostgreSQL instance
- Goldengate Microservices instance

3. [Autonomous Database offerings](https://docs.oracle.com/en-us/iaas/Content/Database/Concepts/adboverview.htm)
- Target ATP

4. [Oracle Cloud Marketplace](https://docs.oracle.com/en-us/iaas/Content/Marketplace/Concepts/marketoverview.htm)
- Goldengate non-oracle deployment 
- Goldengate Microservices deployment

## **Rate this Workshop**

Don't forget to rate this workshop!  We rely on this feedback to help us improve and refine our LiveLabs catalog.  Follow the steps to submit your rating.

1.  Go back to your **workshop homepage** in LiveLabs by going back to your workshop and clicking the Launch button.
2.  Click on the **Brown Button** to re-access the workshop  

    ![](/images/workshop-homepage-2.png " ")

3.  Click **Rate this workshop**

    ![](/images/rate-this-workshop.png " ")

## Acknowledgements

* **Author** - Bilegt Bat-Ochir " Senior Solution Engineer"
* **Contributors** - John Craig "Technology Strategy Program Manager", Patrick Agreiter "Senior Solution Engineer"
* **Last Updated By/Date** - Bilegt Bat-Ochir 3/22/2021