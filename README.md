# Windows 10 Feature Updates Scripts Framework

#### UPDATE: 08/30/2020
I have re-worked this repo to enable you to generate all of the files/scripts then create CIs, Baselines and an Application. Follow the instructions at the end of the script to complete the setup - there are still some manual steps required.

I would recommend downloading the whole repo, then editing the first set of parameters in SetupFUFramework.ps1 to add your server and file share information. You can also pass them in as parameters.

Note: Once the application has been created within ConfigMgr, the script will exit if you attempt to re-run. You must delete the app (or rename in ConfigMgr or in the script parameters) before the script will run again.

## Getting Started
- [ ] Clone the repo
- [ ] Replace `.\Content\Scripts\ADD_SETUPDIAG_HERE.MD` with `SetupDiag.exe` from [https://docs.microsoft.com/en-us/windows/deployment/upgrade/setupdiag](https://docs.microsoft.com/en-us/windows/deployment/upgrade/setupdiag)
- [ ] Run `SetupFramework.ps1` with custom parameters (Example shown in `Usage`)

## Usage
```Powershell
    .\SetupFUFramework -SiteCode "PS1" -SiteServer "CM01.ASD.NET" -ApplicationFolderName "FUApplication" -ApplicationSourceRoot "\\CM01.ASD.NET\Media\$($ApplicationFolderName)" -NetworkLogPath "\\CM01.ASD.NET\FeatureUpdateLogs"
```

## Ouput
### SetupFUFramework.ps1 Script Output
* .\\\<ApplicationFolderName>
  * Scripts
    * Process-Content.ps1
    * Process-FeatureUpdateLogs.ps1
    * Process-SetupDiag.ps1
    * SetupComplete.cmd
    * SetupDiag.exe
    * Trigger-DCMEvaluation.ps1
  * Update
    * failure.cmd
    * postuninstall.cmd
    * precommit.cmd
    * preinstall.cmd
    * success.cmd
* .\Exports
  * Feature Update - Inventory OSVersionHistory and SetupDiag_<CI_ID>.cab
  * Feature Update - No Logged On User Failure_<CI_ID>.cab
  * Exports\Feature Update - Scripts and Files Are Present_<CI_ID>.cab

### In ConfigMgr
* \Software Library\Overview\Application Management\Applications
  * Feature Update - Client Content
* \Assets and Compliance\Overview\Compliance Settings\Configuration Baselines\Feature Updates
  * Feature Update - Inventory OSVersionHistory and SetupDiag
  * Feature Update - No Logged On User Failure
  * Feature Update - Scripts and Files Are Present
* \Assets and Compliance\Overview\Compliance Settings\Configuration Items\Feature Updates
  * Feature Update - Feature Update Files
  * Feature Update - No logged on interactive user - Failed
  * Feature Update - OS Version Inventory
  * Feature Update - SetupConfig.ini
  * Feature Update - SetupDiag Inventory
  * Feature Update - SetupDiag Results
  * Feature Update - SetupDiag Version

### Application Installation
On the client, files are stored in `c:\~FeatureUpdateTemp`.
Logs are written to `c:\~FeatureUpdateTemp\Logs` and `c:\Windows\CCM\Logs`
* C:\~FeatureUpdateTemp
  * Scripts
    * Process-Content.ps1
    * Process-FeatureUpdateLogs.ps1
    * Process-SetupDiag.ps1
    * SetupComplete.cmd
    * SetupDiag.exe
    * Trigger-DCMEvaluation.ps1
* C:\Windows\System32\update\\\<GUID>
    * failure.cmd
    * postuninstall.cmd
    * precommit.cmd
    * preinstall.cmd
    * success.cmd

## Additional Steps Required

After running SetupFUFramework you must do the following:

- [ ] Add a Version detection Rule for SetupDiag.exe to the `Feature Update - SetupDiag Version` Configuration Item.
`Rule Name:      SetupDiag.exe Version = 1.6.0.42`
`Rule Type:      Value`
`Property:       File Version`
`Type            Equals`
`Value:          1.6.0.42`
`NonCompliance : Critical`

- [ ] Import OSVersionHistory.MOF into Default Client Settings Hardware Inventory.
- [ ] Import SetupDiag.MOF into Default Client Settings Hardware Inventory.
- [ ] Distribute Content for the application Feature Update - Client Content.
- [ ] Deploy the application **Feature Update - Client Content** to all Windows 10 devices.
- [ ] Deploy the Configuration Baseline **Feature Update - Inventory OSVersionHistory and SetupDiag** to all Windows 10 devices (_Set to Remediate_).
- [ ] Deploy the Configuration Baseline **Feature Update - No Logged On User Failure** to all Windows 10 devices (_Set to Remediate_).
- [ ] Deploy the Configuration Baseline **Feature Update - Scripts and Files Are Present** to all Windows 10 devices (_Set to Remediate_).
- [ ] Create new Compliant collection for **Feature Update - Scripts and Files Are Present** Baseline Deployment.
- [ ] Create new Feature Update deployment collection using the **Feature Update - Scripts and Files Are Present** Baseline Deployment Compliant collection as the limiting collection.
- [ ] Use this new collection as your limiting collection for all Feature Update deployments. It will ensure that you don't deploy to devices without Feature Update files staged.

## Reporting
Use the Power Bi template in the Reporting folder to report on the custom inventory that is generated after installing the MOFs and deploying the CIs/Baselines.

![OS History](https://raw.githubusercontent.com/AdamGrossTX/Windows10FeatureUpdates/master/Images/PowerBI%20Screenshot-OSHistory.jpg)
![Cumulative OS History](https://raw.githubusercontent.com/AdamGrossTX/Windows10FeatureUpdates/master/Images/PowerBI%20Screenshot-CumulativeOSHistory.jpg)
![Build Type By Day](https://raw.githubusercontent.com/AdamGrossTX/Windows10FeatureUpdates/master/Images/PowerBI%20Screenshot-BuildTypeByDay.jpg)
![SetupDiag Failures](https://raw.githubusercontent.com/AdamGrossTX/Windows10FeatureUpdates/master/Images/PowerBI%20Screenshot-SetupDiagFailures.jpg)
![Computer Details](https://raw.githubusercontent.com/AdamGrossTX/Windows10FeatureUpdates/master/Images/PowerBI%20Screenshot-ComputerDetails.jpg)




## Demos

[Running SetupFUFramework](https://youtu.be/8g3M_ekYvQg)  
[Distributing and Deploying Application](https://youtu.be/9O2SJ4MOmDU)  
[Importing MOFs](https://youtu.be/NlkJBNI8AHw)  
[Updating and Deploying CIs and Baselines](https://youtu.be/sq74eyeNX1E) **Be sure to set the baseline deployments to Remediate. I missed that step in the video.**
