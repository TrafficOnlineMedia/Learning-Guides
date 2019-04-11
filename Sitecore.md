# Sitecore Installation:

Prerequisites:
--------------
1. Sitecore license file
    > Sitecore requires a `license.xml` file in order to run.  If you do not already have a license file, contact your Sitecore partner for one. \
    > Alternatively you can get the one from certified sitecore developer or Company Dev :wink:

2. On-Prem XPO instance packages
    > Only certified developers can download the resources from this site \
    http://dev.sitecore.net

3. Sitecore Installation Framework
    > Download SIF 2 version because it is required for Sitecore version above 9.0+
     https://dev.sitecore.net/Downloads/Sitecore_Installation_Framework/

4. Low Effort Solr Install
    - https://jermdavis.wordpress.com/2017/10/30/low-effort-solr-installs/
    - Installing Solr as a service (`Install-Solr.ps1`)
        - https://gist.github.com/jermdavis
            - https://gist.github.com/jermdavis/8d8a79f680505f1074153f02f70b9105

Step 01:
--------

**Download the Sitecore 9.x (WDP XP0 packages)**

<code>File: **`Sitecore 9.1.0 rev. 001564 (WDP XP0 packages).zip`**</code>

This package have multiple zip files
    
1. `Sitecore 9.1.0 rev. 001564 (OnPrem)_single.scwdp.zip`
2. `Sitecore 9.1.0 rev. 001564 (OnPrem)_xp0xconnect.scwdp.zip`
3. `Sitecore.IdentityServer 2.0.0 rev. 00157 (OnPrem)_identityserver.scwdp.zip`
4. `XP0 Configuration files 9.1.0 rev. 001564.zip`

**Download Low Effort Solr Install Script**
    
https://gist.github.com/jermdavis/8d8a79f680505f1074153f02f70b9105

Open downloaded file and change parameters in the powershell Script
  ```powershell
  # filename: 01_LowEffortSolrInstall.ps1
  $solrVersion = "7.2.1",
  $installFolder = "c:\sitecore\software\solr721",
  $solrPort = "8983",
  $solrHost = "solr",
  $solrSSL = $true,
  $nssmVersion = "2.24",
  $JREVersion = "1.8.0_201"
  ```

Step 02:
--------

- If `solr-7.2.1` service do not start then check `JRE` is installed or not, if not then install the exact version that you have mentioned in the solr installer script. For me this is `$JREVersion = "1.8.0_201"`
- If you have installed the different or latest `JRE` version then go to system setting and modify the Environment Variable `JAVA_HOME` and set the value which version is installed in your system.
- then stop and restart the `solr` service and it should resolve the issue and start the service.
- verify the solr service is running and working properly by going to this link \
    https://solr:8983/solr/

Step 03:
--------

**Install or Upgrade the SIF**

Sitecore Install Framework, Options:

1. Upgrade from version 1
    > if you have already sitecore 9 is installed then you should have version 1 installed in your system then you need to upgrade it to SIF version 2.x
2. Install from repositoy
3. Install from local source

**Install SIF from local source**

- Download the SIF 2 version
    - https://dev.sitecore.net/Downloads/Sitecore_Installation_Framework/
- Extract the zip file
- Run these commands

```powershell
$ cd C:\sitecore\install\SitecoreInstallFramework.2.1.0
$ Import-Module .\SitecoreInstallFramework.psd1
```

If you get the below `SqlServer` Error


<code>Error: **`The script 'Invoke-RemoveSqlDatabaseTask.ps1' cannot be run because the following modules that are specified by the "#requires" statements of the script are missing: SqlServer.`**</code>

![Install-SitecoreConfiguration Issue](/assets/images/sitecore/import-module-issue.jpg)

Re-run the Import module command with `-Verbose` option to check the more information about the error, Normally it will fixed the error by running the command, then try the normal command again and see if error still exist or not.

```bash
$ Import-Module .\SitecoreInstallFramework.psd1 -Verbose
$ Import-Module .\SitecoreInstallFramework.psd1
```

If you still get the error then run this command, the reason we are using `-ExecutionPolicy ByPass` flag is because it will resolve the permission issues

```bash
# if issue still not resolve then see the below link for more information
# https://letsdositecore.wordpress.com/2018/12/04/the-install-sitecoreconfiguration-command-was-found-in-the-module-sitecoreinstallframework-but-the-module-could-not-be-loaded-for-more-information-run-import-module-sitecoreinstallframework/
$ powershell -ExecutionPolicy ByPass Import-Module SitecoreInstallFramework
```

Step 04:
--------

**Configure & Run the Sitecore Installation Script**

- `02_Install-XP0-SingleDeveloper.ps1`
    - This installation script you can get in the <code>**`XP0 Configuration files 9.1.0 rev. 001564.zip`**</code> file and this zip file exist in the main zip file  <code>__Sitecore 9.x (WDP XP0 packages) - `Sitecore 9.1.0 rev. 001564 (WDP XP0 packages).zip`__</code>

- Configure variables in the above script before running
    - Prefix - for multiple instances
    - Install Root
    - Solr URL
    - SQL Server Info

```shell
# filename: 02_Install-XP0-SingleDeveloper.ps1
$Prefix = "sc910" #by default anything, it is the prefix to access the frontend and backend (e.g: http://sc910.sitecore/sitecore , http://sc910.sitecore/sitecore/login)
$SitecoreAdminPassword = "Traffic123"
$SCInstallRoot = "C:\sitecore\install"
$SolrUrl = "https://solr:8983/solr"
$SolrRoot = "C:\sitecore\software\solr721\solr-7.2.1"
$SolrService = "Solr-7.2.1"
$SqlAdminUser = "sa"
$SqlAdminPassword = "Traffic123"
# Sitecore requires a license.xml file in order to run. You can get the one from Sitecore partner.
$LicenseFile = "$SCInstallRoot\license.xml"
```

- The configuration files path that specified in the installation script is `Path = "$SCInstallRoot\XP0-SingleDeveloper.json"` which means its pointing to this folder `C:\sitecore\install` so we should move the files from the `C:\sitecore\install\XP0 Configuration files` folder to install root folder `C:\sitecore\install`
- Now our installation script is ready to go, we just need to run windows powershell with admin privileges and execute the script
```bash
$ cd .\sitecore\install\
$ .\02_Install-XP0-SingleDeveloper.ps1
```

If by running the script the below error will appear then its means SIF module is not imported for the current session or you have closed and reopen the windows powershell which clear/erase the previous session.

<code>Error: **`Install-SitecoreConfiguration : The term 'Install-SitecoreConfiguration' is not recognized as the name of a cmdlet, function, script file, or operable program`**</code>

![Install-SitecoreConfiguration Issue](/assets/images/sitecore/install-sitecore-script-issue.jpg)


If the below error will appear then check that your IIS is started, This error will appear if IIS is not running.

![Install-SitecoreConfiguration Issue](/assets/images/sitecore/sif-installation-issue-02.jpg)


In the end when installation finished, it will provide the sitecore urls, see below screenshots:

![Install-SitecoreConfiguration Issue](/assets/images/sitecore/sitecore-urls.jpg)


**Frontend Url:** http://sc910.sc/  

**Backend Url:** http://sc910.sc/sitecore/admin/


[Complete Learning Resources]: http://sitecore.link/ (The most complete Sitecore knowledge base)
[Master Sitecore]: https://www.youtube.com/user/mastersitecore (Master Sitecore YouTube Channel)
[Sitecore Documentation]: https://doc.sitecore.net/ (Sitecore Documentation)
[Sitecore Dev Docs]: https://doc.sitecore.com/developers/ (Sitecore Developers Documentation)
[Sitecore Stack Exchange]: https://sitecore.stackexchange.com (Sitecore StackExchange Questions/Answers)
[Learning Resources]: https://sitecore.stackexchange.com/questions/12771/how-to-start-development-with-sitecore-commerce-9-sitecore-commerce-9-learning (How to start development with Sitecore - Learning Resources)
[Sitecore Installation Video]: https://www.youtube.com/watch?v=ImfFBHth1o4 (Installing Sitecore - Running SIF Install Scripts)
[Easy Sitecore Installing Guide]: https://www.youtube.com/watch?v=eLjtfXeyYWo (Installing Sitecore 9.1 In a Few Easy Steps)
