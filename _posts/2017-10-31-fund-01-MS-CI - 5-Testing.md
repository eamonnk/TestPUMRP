---
layout: page
title: Continuous Integration
category: Testing
order: 0
---
### Parts Unlimited MRP App Continuous Integration with VSTS Build ###

In this lab, we have an application called Parts Unlimited MRP. We want to set up
Visual Studio Team Services to be able continuously integrate code into the master
branch of code. This means that whenever code is committed and pushed to the
master branch, we want to ensure that it integrates into our code correctly to
get fast feedback. To do so, we are going to be creating a build definition that
will allow us to compile and run unit tests on our code every time a commit is
pushed to Visual Studio Team Services.

### Video ###

<iframe src="https://channel9.msdn.com/Series/Parts-Unlimited-MRP-Labs/Parts-Unlimited-MRP-App-Continuous-Integration/player" width="960" height="540" allowFullScreen frameBorder="0"></iframe>

### Pre-requisites: ###

-   [Completion of the "Set up Parts Unlimited MRP" lab](fund-0-MS-prereq)
-   Project Administrator rights to the Visual Studio Team Services account

### Tasks Overview: ###

**Create a Continuous Integration Build:** In this step, you will create a build definition in Visual Studio Team Services that will be triggered every time a commit is pushed to your repository in Visual Studio Team Services. 
 
### 1. Create a Continuous Integration Build

>NOTE: Ensure that you have an existing PartsUnlimitedMRP team project that also contains the Git repository cloned from GitHub. If not, complete the ["Set up Parts Unlimited MRP"](fund-0-MS-prereq) lab before going through this lab. 

>In this lab, we will be using the Hosted agent located in Visual Studio Team Services. If you would like to use an on-premises cross-platform agent (Azure subscription needed), you can follow instructions for setting an agent up [with this link](https://www.visualstudio.com/en-us/docs/build/admin/index). 

A continuous integration build will give us the ability to automate whether the code
we checked in can compile and will successfully pass any automated tests that we
have created against it. By using an automated build pipeline, we can quickly validate if our code changes have "broken the build" and fix code before it gets to production. 

**1.** Go to your **account’s homepage**: 

	https://<account>.visualstudio.com
	
**2.** Go to your profile page and select your team project, in this instance **PUMRP1**.

![](<../assets/ci2/ProfileHomePage1.png>)

**3.** Click on the **Build and Release** then under **Builds** click on **+ New Definition**.

![](<../assets/ci2/BuildandReleasepage1.png>)

**4.** In the **Select a template** pane choose **Empty**, and  click **Apply**.

![](<../assets/ci2/ChooseBuildTemplate1.png>)

**5.** Ensure the Team Project is selected as the **Repository source**, the appropriate repository (created in the previous step), and select "Hosted" as the **Default agent queue**, then click **Create**.

![](<../assets/ci/build_select_repo.png>)

**6.** Click on the **Build** tab, click **Add build step...**, and then click the **Add** button three times next to the Gradle task to add three Gradle tasks to the script. Gradle will be used to build the Integration Service, Order Service, and Clients components of the MRP app.

![](<../assets/ci/build_add_gradle.png>)

**6.** Select the first Gradle task and **click the pencil icon** to edit the task name. Name the task *IntegrationService* and set the **Gradle Wrapper** to the following location (either type or browse using the **...** button):

	src/Backend/IntegrationService/gradlew

![](<../assets/ci/build_gradle_integration.png>)

**7.** Uncheck the checkbox in **JUnit Test Results** to Publish to TFS/Team Services since we will not be running automated tests in the Integration Service. Expand the **Advanced** section, and set the **Working Directory** to the following location:

	src/Backend/IntegrationService

![](<../assets/ci/build_working_directory_integration.png>)

**8.** Select the second Gradle task and **click the pencil icon** to edit the task name. Name the task *OrderService* and set the **Gradle Wrapper** to the following location:

	src/Backend/OrderService/gradlew

![](<../assets/ci/build_gradle_order.png>)
	
**9.** Since Order Service does have unit tests in the project, we can automate running the tests as part of the build by adding in `test` in the **Gradle tasks** field. Keep the **JUnit Test Results** checkbox to "Publish to TFS/Team Services" checked, and set the **Test Results Files** field to `**/TEST-*.xml`. 

Expand the **Advanced** section, and set the **Working Directory** to the following location:

	src/Backend/OrderService

![](<../assets/ci/build_working_directory_order.png>)

**10.** Select the third Gradle task and **click the pencil icon** to edit the task name. Name the task *Clients* and set the **Gradle Wrapper** to the following location:

	src/Clients/gradlew

![](<../assets/ci/build_gradle_clients.png>)

**11.** Uncheck the checkbox in **JUnit Test Results** to Publish to TFS/Team Services since we will not be running automated tests in Clients. Expand the **Advanced** section, and set the **Working Directory** to the following location:

	src/Clients

![](<../assets/ci/build_working_directory_clients.png>)

**9.** Click **Add build step...**, select the **Utility** tab, and add two **Copy and Publish Build Artifacts** tasks.

![](<../assets/ci/build_add_pub_step.png>)

**10.** Select the first Publish Build Artifacts task, and fill in the input values with the following:

	Copy Root: $(Build.SourcesDirectory)
    Contents: **/build/libs/!(buildSrc)*.?ar
	Artifact Name: drop
	Artifact Type: Server

![](<../assets/ci/build_pub_step_details.png>)

**11.** Select the second Publish Build Artifacts task, and fill in the input values with the following:

	Copy Root: $(Build.SourcesDirectory)
    Contents: **/deploy/SSH-MRP-Artifacts.ps1
			  **/deploy/deploy_mrp_app.sh
              **/deploy/MongoRecords.js
    Artifact Name: deploy
    Artifact Type: Server

![](<../assets/ci/second_copy_publish.png>)

**12.** Go to the **Triggers** tab and **select Continuous Integration (CI)**.

![](<../assets/ci/build_ci_trigger.png>)

**13.** Click **General**, set the default queue to the appropriate queue. Leave as the **Hosted** queue from the previous steps.

![](<../assets/ci/build_general.png>)

**14.** Click **Save**, give the build definition a name (i.e.
*PartsUnlimitedMRP.CI*), and then click **OK**.

![](<../assets/ci/build_save.png>)

**15.** Go to the **Code** tab, select the **index.html** file located at
src/Clients/Web, and click **Edit**.

![](<../assets/ci/edit_index_web.png>)

**16.** Change the &lt;title&gt; on line 5 to **Parts Unlimited MRP** and 
click the **Save button**.
![](<../assets/ci/save_index.png>)

**17.** This should have triggered the build definition we previously created. Click the **Build** tab to see the build in action.
Once complete, you should see build summary similar to this, which includes test results:

![](<../assets/ci/build_summary.png>)

![](<../assets/ci/build_summary_2.png>)

Next steps
----------

In this lab, you learned how to create a Continuous Integration build that runs when new commits are pushed to the master branch. This allows you to get feedback as to whether your changes made breaking syntax changes, or if they broke one or more automated tests, or if your changes are okay. Try these labs out for next steps:

-   [HOL Parts Unlimited MRP Continuous Deployment ](https://github.com/Microsoft/PartsUnlimitedMRP/tree/master/docs/HOL_Continuous-Deployment)

-   [HOL Parts Unlimited MRP Automated Testing](https://github.com/Microsoft/PartsUnlimitedMRP/tree/master/docs/HOL_Automated-Testing)

-   [HOL Parts Unlimited MRP Application Performance Monitoring](https://github.com/Microsoft/PartsUnlimitedMRP/tree/master/docs/HOL_Application-Performance-Monitoring)


# Continuous Feedbacks

#### Issues / Questions about this HOL ??

[If you are encountering some issues or questions during this Hands on Labs, please open an issue by clicking here](https://github.com/Microsoft/PartsUnlimitedMRP/issues)

Thanks
