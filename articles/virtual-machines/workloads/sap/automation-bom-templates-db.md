---
title: Generate Application Database templates for automation
description: How to generate SAP Application Database (SAP DB) templates for use with the deployment automation framework.
author: kimforss
ms.author: kimforss
ms.reviewer: kimforss
ms.date: 10/14/2021
ms.topic: how-to
ms.service: virtual-machines-sap
---

# Generate SAP Application templates for automation

The [SAP deployment automation framework on Azure](automation-deployment-framework.md) uses a Bill of Materials (BoM). Before you can deploy a system using a custom BoM, you need to also create a template for the database. This guide covers SAP Application templates. There's also a [guide for SAP HANA templates](automation-bom-templates-hana.md).

## Prerequisites

- [Get, download, and prepare your SAP installation media and related files](automation-bom-get-files.md) if you haven't already done so. Make sure to have the [name of the SAPCAR utility file that you downloaded](automation-bom-get-files.md#acquire-media) available.
- [Prepare your BoM](automation-bom-prepare.md) if you haven't already done so. Make sure to have the BoM file that you created available.
- An Azure subscription. If you don't already have an Azure subscription, [create a free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- An SAP account with permissions to work with the database you want to use.
- Optionally, create a virtual machine (VM) within Azure to use for transferring SAP media from your storage account. This method improves the transfer speed. Make sure you have connectivity between your VM and the target SAP VM. For example, check that your SSH keys are in place.

## Check media and tools

Before you generate an SAP Application template, make sure you have all required installation media and tools.

1. Sign in to your target VM as the root user.

1. Change the root user password to a known value. You'll use this password later to connect to the SAP Software Provisioning Manager (SWPM).

1. Make and change to a temporary directory.

    ```bash
    mkdir /tmp/workdir; cd $_
    ```

1. Make sure there's a temporary directory for the SAP Application template.

    ```bash
    mkdir /tmp/app_template/
    ```

1. Change the permissions for the SAPCAR utility to make this file executable. Replace `<SAPCAR>.EXE` with the name of the file you downloaded. For example, `SAPCAR_1311-80000935.EXE`.

    ```bash
    chmod +x /usr/sap/install/download_basket/<SAPCAR>.EXE
    ```

1. Make sure the installation folder for SWPM exists.

    ```bash
    mkdir -p /usr/sap/install/SWPM
    ```

1. Extract the SWPM installation file using the SAPCAR utility.

```bash
/usr/sap/install/download_basket/SAPCAR_1311-80000935.EXE -xf /usr/sap/install/SWPM20SP07_0-80003424.SAR -R /usr/sap/install/SWPM/
```

## Install ASCS

You can do an unattended installation of ASCS with a parameter file. This file passes all required parameters to the SWPM installer. 

> [!NOTE]
> To generate the parameter file, you need to partially perform a manual installation. For more information about why, see [SAP NOTE  2230669](https://launchpad.support.sap.com/#/notes/2230669).

1. [Generate your unattended install parameter file for ASCS](#generate-ascs-parameter-file). 
1. [Manually install ASCS using your parameter file](#install-ascs-manually).

## Generate ASCS parameter file

To generate your unattended installation parameter file for ASCS:

1. Sign in to your ASCS VM as the root user through your command-line interface (CLI).

1. Run the command `hostname` to get the host name of the VM from which you're running the installation. Note both the unique hostname (where `<example-vm-hostname>` is in the example output), and the full URL for the GUI.
    
    ```output
    Connecting to the ASCS VM to launch
    ********************************************************************************
    Open your browser and paste the following URL address to access the GUI
    https://<example-VM-hostname>.internal.cloudapp.net:4237/sapinst/docs/index.html
    Logon users: [root]
    ********************************************************************************
    ```

1. [Check that you have all necessary media and tools installed on your ASCS VM](#check-media-and-tools).

1. Launch SWPM as follows. 

    1. Replace `<target-VM-hostname>` with the hostname you previously obtained.

    1. Replace `<XML-stack-file-path>` with the XML stack file path that you created. For example, `/usr/sap/install/config/MP_STACK_S4_2020_v001.xml`.

    ```bash
    /usr/sap/install/SWPM/sapinst                                         \
    SAPINST_XML_FILE=<XML-stack-file-path>.xml    \
    SAPINST_USE_HOSTNAME=<target-VM-hostname>
    
    ```

1. Open your browser and visit the URL for the GUI that you previously obtained.

    1. Accept the security risk warning.
    
    1. Authenticate with your system's root user credentials.
    
1. In the drop-down menu, select **SAP S/4HANA Server 2020** &gt; **SAP HANA Database** &gt; **Installation** &gt; **Application Server ABAP** &gt; **Distributed System** &gt; **ASCS Instance**.
        
1. For **Parameter Mode**, select **Custom**. Then, select **Next**.

1. Configure the SAP system settings:

    1. Make sure the SAP system identifier is `{SID}`.
    
    1. Make sure the SAP mount directory value is `/sapmnt`.
    
    1. Select **Next**.

1. Configure the fully qualified domain name (FQDN) settings: 

    1. Make sure the FQDN value populates automatically.
    
    1. Make sure to enable **Set FQDN for SAP system**.
    
    1. Select **Next**.

1. Set up a main password, which you only use during the creation of this ASCS instance. You can only use alphanumeric characters and the special characters `#`, `$`, `@`, and `_` for your password. You also can't use a digit or underscore as the first character.

    1. Enter a main password.

    1. Confirm the main password.

    1. Select **Next**. 

1. Configure more administrator settings. Other password fields are pre-populated based on the main password you set.

    1. Set the identifier of the administrator OS user (`<sid>adm` where `<sid`> is your SID) to `2000`.

    1. Set the identifier of the SAP system (`sapsys`) to `2000`.

    1. Select **Next**.

1. When prompted for the SAPEXE kernel file path, enter `/usr/sap/install/download_basket`, then select **Next**.

1. Make sure the package status is **Available**, then select **Next**.

1. Make sure the SAP Host Agent installation file status is **Available**, then select **Next**.

1. Provide information for the SAP administrator OS user.

    1. Leave the password as inherited from the main password.

    1. Set the OS user identifier to `2100`.

    1. Select **Next**.

1. Check the installation settings.

    1. Make sure the instance number for the installation is correct.

    1. Make sure to set the virtual host name for the instance.

    1. Select **Next**.

1.  Keep the ABAP message server port settings. These default settings are `3600` and `3900`. Then, `select **Next**.

1. Don't select any other components to install, then select **Next**.

1. Enable **Skip setting of security parameters**, then select **Next**.

1. Enable **Yes, clean up operating system users**, then select **Next**.

1. On **Parameter Summary**, don't do anything yet.

1. In the CLI, find your installation configuration file in the temporary SAP installation directory. At this point, the file is called `inifile.params`.

    1. Run `ls /tmp/sapinst_instdir/` to list the files in the SAP installation directory.

    1. If the file `.lastInstallationLocation` exists, view the file contents and note the directory listed.

    1. If a directory for the product that you're installing exists, such as `S4HANA2020`, go to the product folder. For example, run `cd /tmp/sapinst_instdir/S4HANA2020/CORE/HDB/INSTALL/HA/ABAP/ASCS/`.

1. In your browser, in the SWPM GUI, select **Cancel**. Now, you can do an unattended installation of ASCS.

1. Copy and rename `inifile.params` to `scs.inifile.params` in `/tmp/app_template`. Replace `<path-to-INI-file>` with the path to your INI file as follows:

    ```bash
    cp <path-to-INI-file>/inifile.params /tmp/app_template/scs.inifile.params
    ```

### Install ASCS manually

1. Sign in to your ASCS VM as the root user.

1. Empty all existing files in your temporary work directory, then switch to that directory.

    ```bash
    rm -rf /tmp/workdir/*; cd /tmp/workdir
    ```

1. Launch the ASCS unattended installation. Replace `<target-VM-hostname>` with the ASCS VM host name.

```bash
/usr/sap/install/SWPM/sapinst                                            \
 SAPINST_XML_FILE=/usr/sap/install/config/MP_STACK_S4_2020_v001.xml     \
 SAPINST_USE_HOSTNAME=<target-VM-hostname>                             \
 SAPINST_INPUT_PARAMETERS_URL=/tmp/app_template/scs.inifile.params \
 SAPINST_EXECUTE_PRODUCT_ID=NW_ABAP_ASCS:S4HANA2020.CORE.HDB.ABAPHA     \
 SAPINST_START_GUI=false                                                \
 SAPINST_START_GUISERVER=false
```

## Export SAP file systems

For a distributed system, you must share files between ASCS and the application VMs. These files include the installation media, configuration files, and SID system directory.

Follow the [SAP instructions for exporting directories through Network File Sharing (NFS) for Linux](https://help.sap.com/viewer/e85af73ba3324e29834015d03d8eea84/CURRENT_VERSION/en-US/73297e14899f4dbb878e26d9359f8cf7.html). 

Make sure to export the following directories. Replace `<SID>` with the SID you set during the [generation of the unattended installation parameter file for ASCS](#generate-ascs-parameter-file), such as `{SID}`.

- `/usr/sap/<SID>/system`
- `/usr/sap/install`
- `/tmp/app_template`
- `/sapmnt/<SID>/`

## Mount SAP file systems

To mount the SAP file systems:

1. Connect to the VM as the root user.

1. Check that the mount point exists. Replace `<SID>` with the SID you set.

    ```bash
    mkdir -p /usr/sap/{downloads,install/config,<SID>/SYS} /tmp/app_template /sapmnt/<SID>/{global,profile}
    ```

1. Make sure your exported directories are mounted. Replace `<scs-vm-IP>` with your VM's IP address.

    1. `mount <scs-vm-IP>:/usr/sap/install /usr/sap/install`

    1. `mount <scs-vm-IP>:/usr/sap/install/config /usr/sap/install/config`

    1. `mount <scs-vm-IP>:/usr/sap/<SID>/SYS /usr/sap/<SID>/SYS`
    
    1. `mount <scs-vm-IP>:/tmp/app_template /tmp/app_template`
    
    1. `mount <scs-vm-IP>:/sapmnt/<SID>/global /sapmnt/<SID>/global`
    
    1. `mount <scs-vm-IP>:/sapmnt/<SID>/profile /sapmnt/<SID>/profile`

## Load database content

Make sure the following settings are in place on the PAS VM before you begin:

- Install and configure your HANA and SCS instances. These instances must be online before you complete the database content load.

- The `<sid>adm` user you [created when you generated the unattended installation file for ASCS](#generate-ascs-parameter-file) must be a member of the `sapinst` group.

- The user identifier for `<sid>adm` must match the value of `hdblcm`. This example uses `2000`.

- The SWPM needs access to `/sapmnt/<SID>/global/`. To configure permissions, run `chown <sid>adm:sapsys /sapmnt/<SID>/global`.

### Generate database load template

To generate an unattended installation parameter file for the database content load:

1. Make and change to a temporary directory. Replace `<sid>` with your SID.
    
    ```bash
    sudo install -d -m 0777 <sid>adm -g sapinst "/tmp/db_workdir"; cd $_
    ```

1. Launch the SWPM and note the listed URL.

    ```bash
    /usr/sap/install/SWPM/sapinst   \
    SAPINST_XML_FILE=/usr/sap/install/config/MP_STACK_S4_2020_v001.xml
    ```

1. In your browser, visit the URL you noted.

1. Accept the security risk warning.
    
1. Authenticate with your system's root user credentials.

1. Create a distributed system with custom parameters.

    1. In the drop-down menu, go to **SAP S4/HANA Server 2020** &gt; **SAP HANA Database** &gt; **Installation** &gt; **Application Server ABAP** &gt; **Distributed System** &gt; **Database Instance** &gt; **Distributed System**.

    1. Select the **Custom** parameter mode.

    1. Select **Next**.

1. Note the path of the profile directory that the ASCS installation creates. For example, `/usr/sap/<SID>/SYS/profile` where `<SID>` is your SID. Then, select **Next**.

1. Enter the ABAP message server port for your ASCS instance. The port number is `36<InstanceNumber>`, where `<InstanceNumber>` is the HANA instance number. For example, if there are zero instances, `3600` is the port number. Then, select **Next**.

1. Enter your main password to use during the installation of database content. Then, select **Next**.

1. Make sure the details for the administrator user `<SID>adm` where `SID` is your SID) are correct. Then, select **Next**.

1. Enter your information for the **SAP HANA Database Tenant**. 

    1. For **Database Host**, enter the host name of the HANA database VM. To find this host name, go to the resource page in the Azure portal.

    1. For **Instance Number**, enter the HANA instance number. For example, `00`.

    1. Enter an identifier for the new database tenant. For example, `S4H`.

    1. Keep the automatically generated password for the database system administrator.

    1. Select **Next**.

1. Make sure your connection details are correct. Then, select **OK**.

1. Enter your administrator password for the system database. Then, select **Next**.

1. Enter the path to your SAPEXE kernel, `/usr/sap/install/download_basket`. Then, select **Next**.

1. Review which files are available.
    
    1. Select **Next**.

    1. Make sure the `SAPHOSTAGENT` file is available.

    1. Select **Next** again.

1. On the password confirmation page, select **Next**.

1. Review that all core HANA database export files are available. Then, select **Next**.

1. On **Database Schema** for `DBACOCKPIT`, select **Next**.

1. On **Database Schema** for `SAPHANADB`, select **Next**.

1. On **Secure Storage for Database Connection**, select **Next**.

1. On **SAP HANA Import Parameters**, select **Next**.

1. Enter the password for the HANA database administrator (`<SID>adm`) for the database VM. Then, select **Next**.

1. On **SAP HANA Client Software Installation Path**, select **Next**.

1. Make sure the SAP HANA client file is available. Then, select **Next**.

1. Make sure to enable **Yes, clean up operating system users**. Then, select **Next**.

1. On **Parameter Summary**, don't select anything yet.

1. Open your CLI and find your installation configuration file.

    1. List the files in your temporary directory, `/tmp/sapinst_instdir/`.

    1. Make sure the installation configuration file `inifile.params` is there.

    1. If the file `lastInstallationLocation` is there, open the file. Note the directory listed in the file contents.

    1. If there's already a directory for the product that you're installing, such as `S4HANA2020`, go to the matching folder. For example, `/tmp/sapinst_instdir/S4HANA2020/CORE/HDB/INSTALL/HA/ABAP/DB/`.

1. Open SWPM again.

1. Select **Cancel**. You can now use the unattended method for database content loading.

1. Copy and rename your installation configuration file as follows. Replace `<path_to_config_file>` with the path to your configuration file.

    ```bash
    cp <path_to_config_file>/inifile.params /tmp/app_template/db.inifile.params
    ```

1. Check the version of the `sapinst` tool in SWPM.
    
    ```bash
    /usr/sap/install/SWPM/sapinst -version
    ```

1. If the version of `sapinst` is greater than `749.0.6`, also copy the files `keydb.xml` and `instkey.pkey` to follow [SAP Note 2393060](https://launchpad.support.sap.com/#/notes/2393060). Replace `<path_to_config_file>` with the path to your configuration file.
    
    ```bash
    cp <path_to_config_file>/{keydb.xml,instkey.pkey} /tmp/app_template/
    ```

### Manually load database content

To load the database manually using your template:

1. Connect to your PAS VM as the root user through your CLI.

1. Remove all files from your temporary working directory, then go to that directory.

    ```bash
    rm -rf /tmp/db_workdir/*; cd /tmp/db_workdir
    ```

1. Launch the database load process through SWPM.

    ```bash
    /usr/sap/install/SWPM/sapinst                                           \
    SAPINST_INPUT_PARAMETERS_URL=/tmp/app_templates/db.inifile.params  \
    SAPINST_STACK_XML=/usr/sap/install/config/MP_STACK_S4_2020_v001.xml     \
    SAPINST_EXECUTE_PRODUCT_ID=NW_ABAP_DB:S4HANA2020.CORE.HDB.ABAP          \
    SAPINST_SKIP_DIALOGS=true                                               \
    SAPINST_START_GUI=false SAPINST_START_GUISERVER=false
    ```

## Install PAS 

You can do an unattended installation for ABAP Primary Application Server (PAS) as follows.

1. Make sure you have a fully built HANA database and ASCS instance.

1. [Generate an unattended installation parameter file](#generate-pas-parameter-file).

1. [Install PAS manually using the parameter file](#install-pas-manually).

### Generate PAS parameter file

Generate an unattended installation parameter file for use with PAS. These files all begin with `inifile`. 

> [!IMPORTANT]
> You might not see some of these settings in 2020 versions of SAP products. In that case, skip the step.

1. Connect to your PAS VM through your CLI.

1. [Check that you have all necessary media and tools installed on your PAS VM](#check-media-and-tools).

1. Create and change to a temporary directory. Replace `<SID>` with your SID.

    ```bash
    sudo install -d -m 0777 <SID>adm -g sapinst "/tmp/pas_workdir"; cd $_
    ```
    
1. Connect to the PAS node as the root user. 

1. Sign in to the SWPM. 

    1. Go to the URL for the SWPM GUI. You got this URL when you [generated the unattended installation file for ASCS](#generate-ascs-parameter-file).

    1. Accept the security warning.
    
    1. Authenticate with your system's root user credentials.

1. In the drop-down menu, go to **SAP S/4HANA Server 2020** &gt; **SAP HANA Database** &gt; **Installation** &gt; **Application Server ABAP** &gt; **Distributed System** &gt; **Primary Application Server Instance**.

1. On **Parameter Settings**, select **Custom**. Then, select **Next**.

1. Make sure the **Profile Directory** is set to `/sapmnt/<SID>/profile/` or `/usr/sap/<SID>/SYS/profile`, where `<SID>` is your SID. Then, select **Next**.

1. Set the **Message Server Port** to `36<instance-number>`, where `<instance-number>` is the ASCS instance number. For example, `3600`. Then, select **Next**.

1. Set the main password for all users. Then, select **Next**.

1. Wait for the list `below-the-fold-list` to populate. Then, select **Next**.

1. Make sure to disable the setting **Upgrade SAP Host Agent to the version of the provided SAPHOSTAGENT.SAR archive**. Then, select **Next**.

1. Enter the instance number for the SAP HANA database, and the database system administrator password. Then, select **Next**.

1. On **Configuration of SAP liveCache with SAP HANA**, select **Next**.

1. On **Database Schema** for `DBACOCKPIT`, select **Next**.

1. On **Database Schema** for `SAPHANADB`, select **Next**.

1. On **Secure Storage for Database Connection**, select **Next**.

1. Make sure the PAS instance number and instance host are correct. Then, select **Next**.

1. On **ABAP Message Server Ports**, select **Next**.

1. On **Configuration of Work Processes**, select **Next**.

1. On **ICM User Management for the SAP Web Dispatcher**, select **Next**.

1. On **SLD Destination for the SAP System OS Level**, configure these settings:

    1. Enable **No SLD destination**. Then, select **Next**.

    1. Enable **Do not create Message Server Access Control List**. Then, select **Next**

    1. Enable **Run TMS**. 

    1. Set user password for **TMSADM** int **Client 000** to the main password. Then, select **Next**.

    1. Set **SPAM/SAINT Update Archive** to `/usr/sap/install/config/KD75371.SAR`.

    1. Enable **No for Import ABAP Transports**. Then, select **Next**.


1. On **Preparing for the Software Update Manager**, make sure to enable **Extract the SUM*.SAR Archive**. Then, select **Next**.

1. On **Software Package Browser**, review the table **Detected Packages**. If the individual package location for **SUM 2.0** is empty, set the package path to `/usr/sap/install/config`. Then, select **Next**.

1. Wait for the package location to populate. Then, select **Next**.

1. On **Additional SAP System Languages**, select **Next**. 

1. On **SAP System DDIC Users**, select **Next**.

1. On **Secure Storage Key Generation**, make sure to select **Individual Key**. Then, select **Next**.

1. On the warning screen:

    1. Copy the key identifier and key value. 
    
    1. Store the key identifier and key value securely.

    1. Select **Next**. 

1. For **Clean up operating system users**, select **Yes**. Then, select **Next**.

1. In your CLI, open your temporary directory for the installation. 

1. Make sure there's a copy of the parameters file `inifile.params`. For example, `/tmp/sapinst_instdir/S4HANA2020/CORE/HDB/INSTALL/DISTRIBUTED/ABAP/APP1/inifile.params`.

1. In SWPM, select **Cancel**. You can now install PAS through the unattended method.

1. Copy and rename your PAS parameter file to `pas.inifile.params` in `/tmp/app_template` as follows. Replace `<path_to_config_file>` with the path to your parameter file. 

    ```bash
    cp <path_to_config_file>/inifile.params /tmp/app_template/pas.inifile.params
    ```

1. Create a copy of `pas.inifile.params` and download to your computer or VM.

### Install PAS manually

To install the PAS manually with your parameter file:

1. Connect to PAS as the root user.

1. Delete the files in your temporary directory, then go to that directory.

    ```bash
    rm -rf /tmp/pas_workdir/*; cd /tmp/pas_workdir
    ```

1. Launch your unattended installation of PAS as follows. Replace `<target_vm_hostname>` with the host name of your PAS VM.

    ```bash
    /usr/sap/install/SWPM/sapinst                                                                                         \
    SAPINST_XML_FILE=/usr/sap/install/config/MP_STACK_S4_2020_v001.xml                                                    \
    SAPINST_USE_HOSTNAME=<target_vm_hostname>                                                                             \
    SAPINST_EXECUTE_PRODUCT_ID=NW_ABAP_CI:S4HANA2020.CORE.HDB.ABAP                                                        \
    SAPINST_INPUT_PARAMETERS_URL=/tmp/app_template/pas.inifile.params                                                     \
    SAPINST_START_GUI=false SAPINST_START_GUISERVER=false
    ```

## Install additional application servers

You can do an unattended installation of additional application servers. You can use the parameter file you generate for multiple AAS installations.

1. Make sure you already [installed PAS](#install-pas).

1. Make sure you have a fully build HANA database and ASCS instance.

1. [Generate an unattended parameter file for AAS](#generate-aas-parameter-file).

1. [Install AAS manually using the parameter file](#install-aas-manually).


### Generate AAS parameter file

Generate an unattended installation parameter file for use with AAS. These files all begin with `inifile`. 

> [!IMPORTANT]
> You might not see some of these settings in 2020 versions of SAP products. In that case, skip the step.

1. Connect to your AAS VM through the CLI.

1. [Check that you have all necessary media and tools installed on your AAS VM](#check-media-and-tools).

1. Make sure the group `sapinst` exists.
    
    ```bash
    groupadd -g 2000 sapinst
    ```

1. Create a temporary directory for your installation as follows. Replace `<sid>` with your SID.

    ```bash
    sudo install -d -m 0777 <sid>adm -g sapinst "/tmp/aas_workdir"; cd $_
    ```

1. Sign in to the SWPM. 

    1. Go to the URL for the SWPM GUI. You got this URL when you [generated the unattended installation file for ASCS](#generate-ascs-parameter-file).

    1. Accept the security warning.
    
    1. Authenticate with your system's root user credentials.

1. In the drop-down menu, **SAP S/4HANA Server 2020** &gt; **SAP HANA Database** &gt; **Installation** &gt; **Application Server ABAP** &gt; **High-Availability System** &gt; **Additional Application Server Instance**.

1. On **Parameter Settings**, select **Custom**. Then, select **Next**.

1. Make sure the **Profile Directory** is set to `/sapmnt/<SID>/profile/` or `/usr/sap/<SID>/SYS/profile`, where `<SID>` is your SID. Then, select **Next**.

1. Set **Message Server Port** to `36<instance-number>` where `<instance-number>` is the ASCS instance number. Then, select **Next**.

1. Set the main password for all users. Then, select **Next**.

1. On **Software Package Browser**, set **Search Directory** to `/usr/sap/install/download_basket`. Then, select **Next**.

1. Wait for the list `below-the-fold-list` to populate. Then, select **Next**.

1. Make sure to enable **Upgrade SAP Host Agent to the version of the provided SAPHOSTAGENT.SAR archive**. Then, select **Next**.

1. Enter the instance number of your SAP HANA database and the database system administrator password. Then, select **Next**.

1. On **Configuration of SAP liveCache with SAP HANA**, select **Next**.

1. On **Database Schema** for `DBACOCKPIT`, select **Next**.

1. On **Database Schema** for `SAPHANADB`, select **Next**.

1. On **Secure Storage for Database Connection**, select **Next**.

1. Make sure the AAS instance number and instance host are correct. Then, select **Next**.

1. On **ABAP Message Server Ports**, select **Next**.

1. On **Configuration of Work Processes**, select **Next**.

1. On **ICM User Management for the SAP Web Dispatcher**, select **Next**.

1. On **SLD Destination for the SAP System OS Level**, make sure to enable **No SLD destination**. Then, select **Next**.

1. Enable **Do not create Message Server Access Control List**. Then, select **Next**.

1. Enable **Run TMS**.

1. Set the password for the user **TMSADM** in **Client 000** to the main password. Then, select **Next**.

1. Set **SPAM/SAINT Update Archive** to `/usr/sap/install/config/KD75371.SAR`.

1. Set **Import ABAP Transports** to **No**. Then, select **Next**.

1. On **Preparing for the Software Update Manager Screen**, enable **Extract the SUM*.SAR Archive**. Then, select **Next**.

1. On **Software Package Browser**, select the table **Detected Packages**. If the individual package location for **SUM 2.0** is empty, set the package path to `usr/sap/install/config`. Then, select **Next**.

1. Wait for the package location to populate. Then, select **Next**.

1. On **Additional SAP System Languages**, select **Next**.

1. Make sure to enable **Yes, clean up operating system users**. Then, select **Next**.

1. Through the CLI, check that your temporary directory now has a copy of the parameter file. For example, `/tmp/sapinst_instdir/S4HANA2020/CORE/HDB/INSTALL/AS/APPS/inifile.params`.

1. Copy and rename the file to `aas.inifile.params` in `/tmp/app_template` as follows. Replace `<path_to_inifile>` with the path to your parameter file.

    ```bash
    cp <path_to_inifile>/inifile.params /tmp/app_template/aas.inifile.params
    ```

1. Create a copy of `aas.inifile.params` and download to your computer or VM.

1. In SWPM, select **Cancel**. You can now do the AAS installation through the unattended method.


### Install AAS manually

To install the AAS manually with your parameter file:

1. Connect to your AAS VM as the root user. 

1. Remove all files from your temporary directory and go to that directory. Replace `<target_vm_hostname>` with the host name of your AAS VM.

    ```bash
    rm -rf /tmp/aas_workdir/*; cd /tmp/aas_workdir
    ```

1. Launch your AAS unattended installation as follows. Replace 

    ```bash
    /usr/sap/install/SWPM/sapinst                                                                          \
    SAPINST_XML_FILE=/usr/sap/install/config/MP_STACK_S4_2020_v001.xml                                     \
    SAPINST_USE_HOSTNAME=<target_vm_hostname>                                                              \
    SAPINST_EXECUTE_PRODUCT_ID=NW_DI:S4HANA2020.CORE.HDB.PD                                                \
    SAPINST_INPUT_PARAMETERS_URL=/tmp/app_template/aas.inifile.params                                      \
    SAPINST_START_GUI=false SAPINST_START_GUISERVER=false
    ```

## Combine parameter files

You can combine your parameter files, which all end with `inifile.params`, into one file for the installation process. 

### Create combination file

To create a file that combines all your parameters:

1. If you haven't already, download each parameter file you created (ASCS, PAS, and AAS). You need these files on the computer or VM from which you're working.

1. Make a backup of each parameter file.

1. Create a new combination file. Name this file for the SAP product that you're using. For example, `S4HANA_2020_ISS_v001.inifile.params`.

1. Open the ASCS parameter file (`scs.inifile.params`) in an editor.

1. Copy the header of the ASCS parameter file into the combination file. For example:

    ```yml
    #########################################################################################################################
    #                                                                                                                       #
    # Installation service 'SAP S/4HANA Server 2020 > SAP HANA Database > Installation                                      #
    #   > Application Server ABAP > Distributed System > ASCS Instance', product id 'NW_ABAP_ASCS:S4HANA2020.CORE.HDB.ABAP' #
    #                                                                                                                       #
    #########################################################################################################################
    ```

1. For each `inifile.params` file you have, copy the product identifier line from the header. Then, copy the product identifiers into the header of your combination file. For example:

    ```yml
    #############################################################################################################################################
    #                                                                                                                                           #
    # Installation service 'SAP S/4HANA Server 2020 > SAP HANA Database > Installation                                                          #
    #   > Application Server ABAP > Distributed System > ASCS Instance', product id 'NW_ABAP_ASCS:S4HANA2020.CORE.HDB.ABAP'                     #
    #   > Application Server ABAP > Distributed System > Database Instance', product id 'NW_ABAP_DB:S4HANA2020.CORE.HDB.ABAP'                   #
    #   > Application Server ABAP > Distributed System > Primary Application Server Instance', product id 'NW_ABAP_CI:S4HANA2020.CORE.HDB.ABAP' #
    #   > Additional SAP System Instances > Additional Application Server Instance', product id 'NW_DI:S4HANA2020.CORE.HDB.PD'                  #
    #                                                                                                                                           #
    #############################################################################################################################################
    ```

1. Open your `bom.yml` file in an editor.

1. Copy the sections for `product_ids` into your combination file.

1. For each `inifile.params` file you have, copy the product identifier from the header into the appropriate part of `product_ids`. For example, copy your ASCS to `scs`:

    ```yml
    product_ids:
      scs: "NW_ABAP_ASCS:S4HANA2020.CORE.HDB.ABAP"
      db:  ""
      pas: ""
      aas: ""
      web: ""
    ```

1. Remove any lines that you commented out or left blank.

1. Save your combination file.

### Improve readability

Next, improve the readability of your [combination file](#create-combination-file):

1. Open your combination file in an editor.

1. Sort all lines not in the header.

1. Remove any duplicated lines.

1. Align all the equals signs. For example:

    ```yml
    archives.downloadBasket                             = /usr/sap/install/download_basket
    HDB_Schema_Check_Dialogs.schemaName                 = SAPHANADB
    HDB_Schema_Check_Dialogs.schemaPassword             = MyDefaultPassw0rd
    HDB_Userstore.doNotResolveHostnames                 = x00dx0000l09d4
    ```

1. Separate the lines by prefixes. For example, `NW_CI_Instance.*` and `NW_HDB_DB.*`.

1. Update the following lines to use Ansible variables:

    1. `archives.downloadBasket                             = {{ download_basket_dir }}`

    1. `HDB_Schema_Check_Dialogs.schemaPassword             = {{ password_hana_system }}`
    
    1. `HDB_Userstore.doNotResolveHostnames                 = {{ hdb_hostname }}`
    
    1. `hostAgent.sapAdmPassword                            = {{ password_master }}`
    
    1. `NW_AS.instanceNumber                                = {{ aas_instance_number }}`
    
    1. `NW_checkMsgServer.abapMSPort                        = 36{{ scs_instance_number }}`
    
    1. `NW_CI_Instance.ascsVirtualHostname                  = {{ scs_hostname }}`
    
    1. `NW_CI_Instance.ciInstanceNumber                     = {{ pas_instance_number }}`
    
    1. `NW_CI_Instance.ciMSPort                             = 36{{ scs_instance_number }}`
    
    1. `NW_CI_Instance.ciVirtualHostname                    = {{ pas_hostname }}`
    
    1. `NW_CI_Instance.scsVirtualHostname                   = {{ scs_hostname }}`
    
    1. `NW_DI_Instance.virtualHostname                      = {{ aas_hostname }}`
    
    1. `NW_getFQDN.FQDN                                     = {{ sap_fqdn }}`
    
    1. `NW_GetMasterPassword.masterPwd                      = {{ password_master }}`
    
    1. `NW_GetSidNoProfiles.sid                             = {{ app_sid | upper }}`
    
    1. `NW_HDB_DB.abapSchemaPassword                        = {{ password_master }}`
    
    1. `NW_HDB_getDBInfo.dbhost                             = {{ hdb_hostname }}`
    
    1. `NW_HDB_getDBInfo.dbsid                              = {{ hdb_sid | upper }}`
    
    1. `NW_HDB_getDBInfo.instanceNumber                     = {{ hdb_instance_number }}`
    
    1. `NW_HDB_getDBInfo.systemDbPassword                   = {{ password_hana_system }}`
    
    1. `NW_HDB_getDBInfo.systemid                           = {{ hdb_sid | upper }}`
    
    1. `NW_HDB_getDBInfo.systemPassword                     = {{ password_hana_system }}`
    
    1. `NW_readProfileDir.profileDir                        = /usr/sap/{{ app_sid | upper }}/SYS/profile`
    
    1. `NW_Recovery_Install_HDB.extractLocation             = /usr/sap/{{ hdb_sid | upper }}/HDB{{ hdb_instance_number }}/backup/data/DB_{{ hdb_sid | upper }}`
    
    1. `NW_Recovery_Install_HDB.sidAdmName                  = {{ hdb_sid | lower }}adm`
    
    1. `NW_Recovery_Install_HDB.sidAdmPassword              = {{ password_master }}`
    
    1. `NW_SAPCrypto.SAPCryptoFile                          = {{ download_basket_dir }}/SAPEXE_300-80004393.SAR`
    
    1. `NW_SCS_Instance.instanceNumber                      = {{ scs_instance_number }}`
    
    1. `NW_Unpack.igsExeSar                                 = {{ download_basket_dir }}/igsexe_12-80003187.sar`
    
    1. `NW_Unpack.igsHelperSar                              = {{ download_basket_dir }}/igshelper_17-10010245.sar`
    
    1. `NW_Unpack.sapExeDbSar                               = {{ download_basket_dir }}/SAPEXEDB_300-80004392.SAR`
    
    1. `NW_Unpack.sapExeSar                                 = {{ download_basket_dir }}/SAPEXE_300-80004393.SAR`
    
    1. `NW_SCS_Instance.scsVirtualHostname                  = {{ scs_hostname }}`
    
    1. `nwUsers.sapadmUID                                   = {{ sapadm_uid }}`
    
    1. `nwUsers.sapsysGID                                   = {{ sapsys_gid }}`
    
    1. `nwUsers.sidadmPassword                              = {{ password_master }}`
    
    1. `nwUsers.sidAdmUID                                   = {{ sidadm_uid }}`
    
    1. `storageBasedCopy.hdb.instanceNumber                 = {{ hdb_instance_number }}`
    
    1. `storageBasedCopy.hdb.systemPassword                 = {{ password_hana_system }}`

### Upload combination file

Finally, upload your combined template file to your SAP Library.

1. Sign in to the [Azure portal](https://portal.azure.com).

1. Select or search for **Storage accounts**.

1. Select the storage account for your SAP Library.

1. In the storage account menu, under **Data storage**, select **Containers**.

1. Select the `sapbits` container.

1. Go to the product folder for your BoM in `sapbits`. For example, `boms/S4HANA_2020_ISS_v001`.

1. If you don't already have a directory called **templates**, create this directory.

1. Open the **templates** directory.

1. Select **Upload**.

1. In the pane, select **Select a file**.

1. Select the combined template file.  For example, `S4HANA_2020_ISS_v001.inifile.params`.

1. Select **Upload**.

## Update BoM with templates

After [combining your parameter files](#combine-parameter-files), update your BoM with the new template files.

1. Open `bom.yml`.

1. In the section `templates`, add your new template file names.  For example:

    ```yml
    templates:
      - name:     "S4HANA_2020_ISS_v001 ini file"
        file:     S4HANA_2020_ISS_v001.inifile.params
        override_target_location: "{{ target_media_location }}/config"
    ```

1. If you're using the scripted application BoM preparation, remove the `#` before the template.

1. Save your changes.

Then, upload the new BoM file to your SAP Library.

1. Sign in to the [Azure portal](https://portal.azure.com).

1. Select or search for **Storage accounts**.

1. Select the storage account for your SAP Library.

1. In the storage account menu, under **Data storage**, select **Containers**.

1. Select the `sapbits` container.

1. Go to the product folder for your BoM in `sapbits`. For example, `boms/S4HANA_2020_ISS_v001`.

1. Open the `boms` directory.

1. Select **Upload**.

1. In the pane, select **Select a file**.

1. Select your BoM file, `bom.yml`, from your computer or VM.

1. Make sure to enable **Overwrite if files already exist**.

1. Select **Upload**.

## Next steps

> [!div class="nextstepaction"]
> [How to generate SAP HANA BoM](automation-bom-templates-hana.md)
