[![Build Status](https://dev.azure.com/mseng/AzureDevOps/_apis/build/status/Obsolete/Cross-Platform/AppStoreExtension.CI?branchName=master)](https://dev.azure.com/mseng/AzureDevOps/_build/latest?definitionId=4518&branchName=master)

<table style="width: 100%; border-style: none;"><tr>
<td width="140px" style="text-align: center;"><img src="apple_default.png" style="max-width:100%" /></td>
<td><strong>Azure DevOps Extension for the Apple App Store</strong><br />
<i>Provides build/release tasks that enable performing continuous delivery to Apple's App Store from an automated Azure DevOps build or release pipeline</i><br />
<a href="https://marketplace.visualstudio.com/items/ms-vsclient.app-store">Install now!</a>
</td>
</tr></table>

# Azure DevOps Extension for the Apple App Store

This extension contains a set of deployment tasks which allow you to automate the release and promotion of app updates to Apple's App Store from your CI environment. This can reduce the effort needed to keep your beta and production deployments up-to-date, since you can simply push changes to the configured source control branches, and let your automated build take care of the rest.

## Prerequisites

* In order to automate the release of app updates to the App Store, you need to have manually released at least one version of the app beforehand.
* The tasks install and use [fastlane](https://github.com/fastlane/fastlane) tools. fastlane requires Ruby 2.0.0 or above and recommends having the latest Xcode command line tools installed on the MacOS computer. 

## Quick Start

Once you have created or retrieved credentials for your App Store account, perform the following steps to automate releasing updates from an Azure DevOps build or release pipeline:

1. Install the App Store extension from the [Azure DevOps Marketplace](https://marketplace.visualstudio.com/items/ms-vsclient.app-store).

2. Go to your Azure DevOps or TFS project, click on the **Pipelines** tab, and create a new pipeline (the "+" icon) that is hooked up to your project's appropriate source repository.

3. Click **Add build step...** and select the necessary tasks to generate your release assets (e.g. **Gulp**, **Cordova Build**).

4. Click **Add build step...** and select **App Store Release** from the **Deploy** category.

5. Configure the **App Store Release** task with the desired authentication method, the generated IPA file path, and the desired release track.

6. Click the **Queue Build** button or push a change to your configured repository in order to run the newly defined build.

7. Your app changes will now be automatically published to the App Store!

## Configuring Your App Store Publisher Credentials

In addition to specifying your publisher credentials directly within each build task, you can also configure your credentials globally and refer to them within each build or release pipeline as needed. To do this, perform the following steps:

1. Setup an Apple developer account (https://developer.apple.com/).

2. Go into your Azure DevOps or TFS project and click on the gear icon in the upper right corner.

3. Click on the **Services** tab.

4. Click on **New service connection** and select **Apple App Store**.

5. Give the new connection a name and enter the credentials for your Apple developer account.

6. Select this connection using the name you chose in the previous step whenever you add either the **App Store Release** or **App Store Promote** tasks to a build or release pipeline.

### Two-Factor Authentication

Apple authentication is region specific, and Microsoft-hosted agents may not be in the same region as your developer machine. Instead, we recommend that you create a separate Apple ID with a strong password and restricted access.  See [this](https://docs.fastlane.tools/best-practices/continuous-integration/#separate-apple-id-for-ci) link for more details.

To use two-factor authentication, you need to setup the `Fastlane Session` variable on the Apple App Store service connection. 

1. Create the fastlane session token by following these [instructions](https://docs.fastlane.tools/best-practices/continuous-integration/#use-of-application-specific-passwords-and-spaceauth).

2. Set this value on the Apple App Store service connection.

#### Use of application specific apple id
If you want to upload apps to TestFlight without trigerring two-factor authentication, you need to setup the App specific apple Id. This value should be taken from Apple ID property in the App Information section in App Store Connect (number). 
The following conditions are required:
1. Application specific apple id should be provided (number)
2. shouldSkipWaitingForProcessing: true
3. isTwoFactorAuth: true (for service connection - you don't need to specify it if app specific password is specified)
4. releaseNotes shouldn't be specified


## Task Reference

In addition to the custom service connection, this extension also contributes the following build and release tasks:

* [App Store Release](#app-store-release) - Allows automating the release of updates to existing iOS TestFlight beta apps or production apps in the App Store.

* [App Store Promote](#app-store-promote) - Allows automating the promotion of a previously submitted app from iTunes Connect to the App Store.

### App Store Release

Allows you to release updates to your iOS TestFlight beta app or production app on the App Store, and includes the following options:

![Release task](/images/release-task-with-advanced.png)

1. **Username and Password** or **Service Connection** - The credentials used to authenticate with the App Store. Credentials can be provided directly or configured via a service connection that can be referenced from the task (via the `Service Connection` authentication method).

2. **Bundle ID** *(String)* - Unique app identifier (e.g. com.myapp.etc).  The **Bundle ID** is only required if "Track" is *Production*.

3. **Binary Path** *(File path, Required)* - Path to the IPA file you want to publish to the specified track.  A glob pattern can be used but it must resolve to exactly one IPA file.

#### Release Options

**Track** *(String, Required)* - Release track to publish the binary to (e.g. `TestFlight`  or `Production` ).

##### Release Options for TestFlight track

1. **What to Test?** *(File path)* - Path to the file containing notes on what to test for this release.

2. **App Specific Apple Id** *(String)* - App specific apple Id allows you to upload applications to a TestFlight track without triggering 2FA. This value should be taken from Apple ID property in the App Information section in App Store Connect.

3. **Skip Build Processing Wait** *(Checkbox)* - Skip waiting for App Store to finish the build processing.

4. **Skip Submission** *(Checkbox)* - Upload a beta app without distributing to testers.

5. **Distribute to External Testers** *(Checkbox)* - Select to distribute the build to external testers (cannot be used with 'Skip Build Processing Wait' and 'Skip Submission').  Using this option requires setting release notes in 'What to Test?'.

6. **Groups** *(String)* - Optionally specify the group(s) of external testers this build should be distributed to. To specify multiple groups, separate group names by commas e.g. 'External Beta Testers,TestVendors'. If not specified the default 'External Testers' is used.

##### Release Options for Production track

1. **Skip Binary Upload** *(Checkbox)* - Skip binary upload and only update metadata and screenshots. Please note that with enabling this option you also need to pass --description or --pkg as additional fastlane parameters.

2. **Upload Metadata** *(Checkbox)* - Upload app metadata to the App Store (e.g. title, description, changelog).

3. **Metadata Path** *(File path)* - Path to the metadata to publish. Expects a format similar to fastlane’s [deliver tool](https://github.com/fastlane/fastlane/tree/master/deliver#readme) which is summarized below:
 
```
$(Specified Directory)
   ├ copyright.txt
   ├ $(languageCodes)
   |    ├ description.txt
   |    ├ keywords.txt
   |    ├ marketing_url.txt
   |    ├ name.txt
   |    ├ privacy_url.txt
   |    ├ release_notes.txt
   |    ├ support_url.txt
   |    ├ subtitle.txt
   |    ├ apple_tv_privacy_policy.txt
   |    ├ promotional_text.txt
   ├ review_information
   |    ├ review_first_name.txt
   |    ├ review_last_name.txt
   |    ├ review_phone_number.txt
   |    ├ review_email.txt
   |    ├ review_demo_user.txt
   |    ├ review_demo_password.txt
   |    ├ review_notes.txt
   ├ primary_category.txt
   ├ secondary_category.txt
   ├ primary_first_sub_category.txt
   ├ primary_second_sub_category.txt
   ├ secondary_first_sub_category.txt
   ├ secondary_second_sub_category.txt
```
More information about metadata folder options you can find [here](http://docs.fastlane.tools/actions/deliver/#available-metadata-folder-options).


4. **Upload Screenshots** *(Checkbox)* - Upload screenshots of the app to the App Store.

5.  **Screenshots Path** *(File path)* - Path to the screenshots to publish.

6. **Submit for Review** *(Checkbox)* - Automatically submit the new version for review after the upload is completed.

7. **Release Automatically** *(Checkbox)* - Automatically release the app once it is approved.

#### Advanced Options

1. **Team Id** *(String)* - The ID of the producing team. Only necessary when in multiple teams.

2. **Team Name** *(String)* - The name of the producing team. Only necessary when in multiple teams.

3. **Install fastlane** *(Checkbox)* - By default, install a version of the [fastlane](https://github.com/fastlane/fastlane) tools.  Uncheck if your build machine already has the version of fastlane to use.

4. **fastlane Version** - **Latest Version** or **Specific Version**.  If *Specific Version* is chosen, you must provide a value for *fastlane Specific Version*.

5. **fastlane Specific Version** *(String)* - The version of fastlane to install (e.g., 2.15.1).

6. **Additional fastlane arguments** *(String)* - Any additional arguments to pass to the fastlane command.

### App Store Promote

Allows you to promote an app previously updated to iTunes Connect to the App Store, and includes the following options:

![Promote task](/images/promote-task-with-advanced.png)

1. **Username and Password** or **Service Connection** - The credentials used to authenticate with the App Store. Credentials can be provided directly or configured via a service connection that can be referenced from the task (via the `Service Connection` authentication method).

2. **Bundle ID** *(String, required)* - The unique identifier for the app to be promoted.

3. **Choose Build** - `Latest` or `Specify build number`. By default the latest build will be submitted for review.

4. **Build Number** - Required if `Specify build number` option is selected in **Choose Build** above. The build number in iTunes Connect that you wish to submit for review.

5. **Release Automatically** *(Checkbox)* - Check to automatically release the app once the approval process is completed.

#### Advanced Options

1. **Team Id** *(String)* - The ID of the producing team. Only necessary when in multiple teams.

2. **Team Name** *(String)* - The name of the producing team. Only necessary when in multiple teams.

3. **Install fastlane** *(Checkbox)* - By default, install a version of the [fastlane](https://github.com/fastlane/fastlane) tools.  Uncheck if your build machine already has the version of fastlane to use.

4. **fastlane Version** - **Latest Version** or **Specific Version**.  If *Specific Version* is chosen, you must provide a value for *fastlane Specific Version*.

5. **fastlane Specific Version** *(String)* - The version of fastlane to install (e.g., 2.15.1).  If a specific version of fastlane is installed, all previously installed versions will be uninstalled beforehand.

6. **Additional fastlane arguments** *(String)* - Any additional arguments to pass to the fastlane command.

## Firewall Issues

The [fastlane](https://github.com/fastlane/fastlane) tools use the iTunes Transporter to upload metadata and binaries. In case you are behind a firewall, you can specify a different transporter protocol injecting in your release pipeline a variable:
`DELIVER_ITMSTRANSPORTER_ADDITIONAL_UPLOAD_PARAMETERS="-t DAV"`
![Fix Firewall issues](/images/variable-definition-firewall-issues.png)

## Support
Support for this extension is provided on our [GitHub Issue Tracker](https://github.com/Microsoft/app-store-vsts-extension/issues).  You
can submit a [bug report](https://github.com/Microsoft/app-store-vsts-extension/issues/new), a [feature request](https://github.com/Microsoft/app-store-vsts-extension/issues/new)
or participate in [discussions](https://github.com/Microsoft/app-store-vsts-extension/issues).

## Contributing to the Extension
See the [developer documentation](CONTRIBUTING.md) for details on how to contribute to this extension.

## Code of Conduct
This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Privacy Statement
The [Microsoft Visual Studio Product Family Privacy Statement](http://go.microsoft.com/fwlink/?LinkId=528096&clcid=0x409)
describes the privacy statement of this software.

Apple and the Apple logo are trademarks of Apple Inc., registered in the U.S. and other countries. App Store is a service mark of Apple Inc.
