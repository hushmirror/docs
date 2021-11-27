# Creating SilentDragon MSI installer for Windows with msitools and wixl
This documentation is for creating a SilentDragon MSI installer for Windows using msitools and wixl. Documentation for msitools is here: https://wiki.gnome.org/msitools Documentation for wixl can refer to WiX toolset as it tries to share the same syntax: https://wixtoolset.org/documentation/manual/v3/ 
This was tested on Debian 11 (Bullseye)
## Install msitools and wixl
`
sudo apt-get -y install msitools wixl
`
## Download latest SilentDragon Windows release and unzip
Latest releases located here: https://git.hush.is/hush/SilentDragon/releases Example below is for SilentDragon 1.3.0
`
wget https://git.hush.is/attachments/a95d4c43-b662-49bc-a3af-a6fbdd4b9724
unzip a95d4c43-b662-49bc-a3af-a6fbdd4b9724
`
This will unzip into a directory named SilentDragon-1.3.0-win
`
cd SilentDragon-1.3.0-win
`
## Copy silentdragon.ico to SilentDragon directory
The icon file `silentdragon.ico` is currently not included in release zip, but is required to create an MSI installer with a smaller size. Setting icon SourceFile in the .wxs file to silentdragon.exe will duplicate the .exe and make the installer over 100MB instead of ~80MB. How you get this .ico file is up to you. I extracted from .exe on Windows. Maybe it will be included in .zip release in the future to make this easier for anyone trying to create a MSI file for the first time. 

## Create WiX source file (.wxs)
This is an XML file. You should name it what you want the .msi file to be named IE: `SilentDragon-1.3.0.wxs`
Below is a copy of the contents of the `SilentDragon-1.3.0.wxs file` Note that GUID's all need to be unique. This will create shortcuts to SilentDragon on desktop and in the start menu as well as have uninstall support from Add/Remove Programs. A tutorial that explains the basics of creating these files can be found here: https://www.firegiant.com/wix/tutorial/
```
<?xml version='1.0' encoding='windows-1252'?>
<Wix xmlns='http://schemas.microsoft.com/wix/2006/wi'>
  <Product Name='SilentDragon' Id='c8fae41c-889b-4621-b343-76491cd91b1c' UpgradeCode='a41d7a93-a6cb-41fb-89d2-706a8c22bc99'
    Language='1033' Codepage='1252' Version='1.3.0' Manufacturer='HUSH'>

    <Package Id='*' Keywords='Installer' Description="HUSH SilentDragon Installer"
      Comments='' Manufacturer='HUSH'
      InstallerVersion='100' Languages='1033' Compressed='yes' SummaryCodepage='1252' />

    <Media Id='1' Cabinet='SilentDragon.cab' EmbedCab='yes' DiskPrompt="CD-ROM #1" />
    <Property Id='DiskPrompt' Value="HUSH SilentDragon 1.3.0 Installation [1]" />

    <Directory Id='TARGETDIR' Name='SourceDir'>
      <Directory Id='ProgramFilesFolder' Name='PFiles'>
        <Directory Id='HUSH' Name='HUSH'>
          <Directory Id='INSTALLDIR' Name='SilentDragon'>
            <Component Id='MainExecutable' Guid='56443570-635d-48e4-8448-8ffd0d7c415a'>
              <File Id='SilentDragonEXE' Name='silentdragon.exe' DiskId='1' Source='silentdragon.exe' KeyPath='yes'>
                <Shortcut Id="startmenuSilentDragon" Directory="ProgramMenuDir" Name="SilentDragon" WorkingDirectory='INSTALLDIR' Icon="silentdragon.exe" IconIndex="0" Advertise="yes" />
                <Shortcut Id="desktopSilentDragon" Directory="DesktopFolder" Name="SilentDragon" WorkingDirectory='INSTALLDIR' Icon="silentdragon.exe" IconIndex="0" Advertise="yes" />
              </File>
            </Component>
            <Component Id="asmap" Guid="190ad39b-44fa-4b22-94ee-d42aca7acc7b">
                <File Id="asmap.dat" DiskId='1' Source="asmap.dat" KeyPath="yes"/>
            </Component>
            <Component Id="hush-cli" Guid="e19e8fd8-aeb9-4dad-99bd-70da0a0aa92c2">
                <File Id="hush-cli.exe" DiskId='1' Source="hush-cli.exe" KeyPath="yes"/>
            </Component>
            <Component Id="hush-tx" Guid="f96b2a39-4734-4a8d-abc3-895006052e97">
                <File Id="hush-tx.exe" DiskId='1' Source="hush-tx.exe" KeyPath="yes"/>
            </Component>
            <Component Id="hushd" Guid="fa66588f-8788-4b29-b6d6-c4a903e49d79">
                <File Id="hushd.exe" DiskId='1' Source="hushd.exe" KeyPath="yes"/>
            </Component>
            <Component Id="READMEFILE" Guid="b20b11de-ded8-4f3c-9797-9bc72c70b42a">
                <File Id="README" DiskId='1' Source="README" KeyPath="yes"/>
            </Component>
            <Component Id="sapling-output" Guid="1ecc1590-ddf3-4f6d-94c5-6bf091aef77a">
                <File Id="sapling-output.params" DiskId='1' Source="sapling-output.params" KeyPath="yes"/>
            </Component>
            <Component Id="sapling-spend" Guid="828d3827-5f2b-47c6-8717-8a664054a2af">
                <File Id="sapling-spend.params" DiskId='1' Source="sapling-spend.params" KeyPath="yes"/>
            </Component>			
          </Directory>
        </Directory>
      </Directory>

      <Directory Id="ProgramMenuFolder" Name="Programs">
        <Directory Id="ProgramMenuDir" Name="SilentDragon">
          <Component Id="ProgramMenuDir" Guid="7ac6af1f-1377-4158-915f-c410cc5cd2a9">
            <RemoveFolder Id='ProgramMenuDir' On='uninstall' />
            <RegistryValue Root='HKCU' Key='Software\[Manufacturer]\[ProductName]' Type='string' Value='' KeyPath='yes' />
          </Component>
        </Directory>
      </Directory>

      <Directory Id="DesktopFolder" Name="Desktop" />
    </Directory>

    <Feature Id='Complete' Level='1'>
      <ComponentRef Id='MainExecutable' />
      <ComponentRef Id='ProgramMenuDir' />
      <ComponentRef Id='asmap' />
      <ComponentRef Id='hush-cli' />
      <ComponentRef Id='hush-tx' />
      <ComponentRef Id='hushd' />
      <ComponentRef Id='READMEFILE' />
      <ComponentRef Id='sapling-output' />
      <ComponentRef Id='sapling-spend' />
    </Feature>

    <Icon Id="silentdragon.exe" SourceFile="silentdragon.ico" />

  </Product>
</Wix>
```
## Build MSI file from WiX source file
`
wixl -v SilentDragon-1.3.0.wxs
`
This will create a `SilentDragon-1.3.0.msi` file in the same directory which can then be distributed. If it created without error and is larger than 0kb, congratulations!