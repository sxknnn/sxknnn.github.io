# Windows Installer

For development a virtual machine and vagrant file are used to host the bell app nation, but in deployments the necessary software is installed directly to the machine. Installing all the software and then setting up the database is tedious, and may be difficult for someone not as familiar with computers. A packaged installer file that does everything for you would be helpful for such people, and is the aim of this project.

## Inno Setup

Inno Setup is a program for creating installation programs for Windows. You can use the GUI to create your installer, or you can edit its simple file format directly. Because the GUI is windows only, and mostly self-explanatory,  this document will just focus on explaining the file itself.

Currently the installer definition file resides on the `inno` branch on `ole-vi/BeLL-Apps`.  You can view the whole file [here](https://github.com/ole-vi/BeLL-Apps/blob/inno/installer/BeLL-Apps-Installer.iss). To help you understand how it works, it will be divided up into sections and explained.

```
; Script generated by the Inno Script Studio Wizard.
; SEE THE DOCUMENTATION FOR DETAILS ON CREATING INNO SETUP SCRIPT FILES!

#define MyAppName "BeLL-App"
#define MyAppVersion "0.12.67"
#define MyAppPublisher "Open Learning Exchange, International"
#define MyAppURL "http://www.ole.org"
```
At the start of the document, you can see some comments, which start with `;` as well as the declaration of some constants. These constant variables are used later in the script. On to the first section, called `Setup`
```
[Setup]
; NOTE: The value of AppId uniquely identifies this application.
; Do not use the same AppId value in installers for other applications.
; (To generate a new GUID, click Tools | Generate GUID inside the IDE.)
AppId={{F712730B-0D52-4BBA-9521-5A105731005D}}
AppName={#MyAppName}
AppVersion={#MyAppVersion}
;AppVerName={#MyAppName} {#MyAppVersion}
AppPublisher={#MyAppPublisher}
AppPublisherURL={#MyAppURL}
AppSupportURL={#MyAppURL}
AppUpdatesURL={#MyAppURL}
DefaultDirName={pf}\{#MyAppName}
DefaultGroupName={#MyAppName}
DisableProgramGroupPage=yes
OutputDir=".\"
OutputBaseFilename=Bell-0.12.67-setup

Compression=lzma
SolidCompression=yes
```
Here we supply global settings for the installer and metadata for our application. The AppId is important, as that is how inno setup tells your app apart from other apps installed by inno setup. You can see all the fields you are allowed to set in this section [here](http://www.jrsoftware.org/ishelp/topic_setupsection.htm).

Next, the languages and files sections:
```
[Languages]
Name: "english"; MessagesFile: "compiler:Default.isl"

[Files]
; NOTE: Don't use "Flags: ignoreversion" on any shared system files
Source: "curl.exe"; DestDir: "{app}"; Flags: ignoreversion deleteafterinstall
;Source: "x64\apache-couchdb-2.0.0.1.msi"; DestDir: "{tmp}"; Flags: ignoreversion
Source: "x64\node-v6.9.2-x64.msi"; DestDir: "{tmp}"; Flags: ignoreversion
Source: "dbsetup.bat"; DestDir: "{app}"; Flags: deleteafterinstall
Source: "Bell_logo.ico"; DestDir: "{app}"
Source: "x64\setup-couchdb-1.6.1_R16B02.exe"; DestDir: "{tmp}"; Flags: ignoreversion
Source: "uninstall_nodejs.bat"; DestDir: "{app}"
Source: "..\.idea\*"; DestDir: "{app}\.idea"; Flags: ignoreversion createallsubdirs recursesubdirs
Source: "..\app\*"; DestDir: "{app}\app"; Flags: ignoreversion createallsubdirs recursesubdirs
Source: "..\attachments\*"; DestDir: "{app}\attachments"; Flags: ignoreversion createallsubdirs recursesubdirs
Source: "..\databases\*"; DestDir: "{app}\databases"; Flags: ignoreversion createallsubdirs recursesubdirs
Source: "..\docs\*"; DestDir: "{app}\docs"; Flags: ignoreversion createallsubdirs recursesubdirs
Source: "..\images\*"; DestDir: "{app}\images"; Flags: ignoreversion createallsubdirs recursesubdirs
Source: "..\init_docs\*"; DestDir: "{app}\init_docs"; Flags: ignoreversion createallsubdirs recursesubdirs
Source: "..\node_modules\*"; DestDir: "{app}\node_modules"; Flags: ignoreversion createallsubdirs recursesubdirs
Source: "..\update_nation\*"; DestDir: "{app}\update_nation"; Flags: ignoreversion createallsubdirs recursesubdirs
```
The language is English, and we want the default text supplied by inno setup. For the file section `Source` specifies where the file is located *right now* so that inno setup can include it in the package, and `DestDir` specifies where the file should be installed to when the installer is actually run. There are built in constants that you can use in files paths, such as `{app}` and `{tmp}`, which are the installation folder and temporary folder respectively. Using a `*` at the end of a source path will include all files in that folder. Using the flag `recursesubdirs` will make inno setup include everything in the sub-directories when using the wildcard `*`.  Finally `createallsubdirs` will make it copy empty directories as well. You can see all of the options for the `Files` section [here](http://www.jrsoftware.org/ishelp/topic_filessection.htm).

```
[Icons]
Name: "{group}\{cm:UninstallProgram,{#MyAppName}}"; Filename: "{uninstallexe}"
Name: "{userdesktop}\MyBell"; Filename: "http://127.0.0.1:5984/apps/_design/bell/MyApp/index.html"; IconFilename: "{app}\Bell_logo.ico"

[Run]
Filename: "msiexec.exe"; Parameters: "/i ""{tmp}\node-v6.9.2-x64.msi"" /passive"
Filename: "{tmp}\setup-couchdb-1.6.1_R16B02.exe"; Parameters: "/SILENT"
;Filename: "msiexec.exe"; Parameters: "/i ""{tmp}\apache-couchdb-2.0.0.1.msi"" /passive"
;Filename: "C:\CouchDB\bin\couchdb.cmd"; Flags: runhidden
Filename: "{app}\dbsetup.bat"; Flags: runhidden

[UninstallRun]
Filename: "C:\Windows\SysWOW64\msiexec.exe"; Parameters: "/x {{EBF9E075-7642-489B-B557-992F349CFB40}} /passive"
Filename: "{pf32}\Apache Software Foundation\CouchDB\unins000.exe"; Parameters: "/SILENT"
Filename: "{app}\uninstall_nodejs.bat"; Flags: runhidden

[UninstallDelete]
Type: files; Name: "{app}\install_log.txt"
```
Here we create an uninstall icon and a desktop icon to open the bell apps. In the run sections you can run scripts and programs. We are installing the other software needed to run the bell app nation here, node, couchdb, and the database setup scripts. Notice `couchdb 2.0` and `couchdb.cmd` are commented out, as they are not used. Note that if you don't pass arguments to your programs and scripts here telling them to run hidden they will pop up during the installation process, which might confuse users. Likewise `UninstallRun` is the programs and scripts to execute when the bell apps are uninstalled. The `UninstallDelete` section defines any additional files or directories you want the uninstaller to delete, **besides** those that were installed/created using `[Files]` or `[Dirs]` section entries.

Finally there is the `[Code]` section:
```Pascal
[Code]
function InitializeSetup(): Boolean;
var
  ErrorCode: Integer;
  JavaVer : String;
  Result1 : Boolean;
begin
    Result := false;
    if RegKeyExists(HKEY_CURRENT_USER, 'Software\Mozilla\Mozilla Firefox') OR RegKeyExists(HKEY_CURRENT_USER, 'Software\Mozilla\Firefox') then
    begin
      Result := true;
    end;

    if Result = false then
    begin
        Result1 := MsgBox('This program requires Firefox. Please download and install Firefox and run this setup again.' + #13 + #10 + 'Would you like to download it now?',
          mbConfirmation, MB_YESNO) = idYes;
        if Result1 = true then
        begin
            ShellExec('open',
              'https://www.mozilla.org/en-US/firefox/new/',
              '','',SW_SHOWNORMAL,ewNoWait,ErrorCode);
        end;
    end;
end;
```
You can write Pascal code in this section. `InitializeSetup` is called before the installation process begins. Here it is used to check if Firefox is installed. If `InitializeSetup` returns false the setup is aborted. You can read about Pascal scripting for inno setup [here.](http://www.jrsoftware.org/ishelp/topic_scriptintro.htm) Most things can be done without any scripting, but only using the sections discussed above.

## Building on Travis
Inno setup only runs on windows (its for creating windows installers after all). But our travis builds are on linux, so if we want to build an installer via Travis we will need [wine](https://www.winehq.org/docs/wineusr-guide/what-is-wine).

We want Travis to build an installer when someone tags a release on Github, so in the `deploy` section of the travis yml we set `provider` to  `releases`.

The relevant sections of the `travis.yml`, with comments added to explain:

```yml
before_install:
  # <snip>
  # This is needed for wine + inno
  - sudo dpkg --add-architecture i386
  # install updates, innoextract, wine, and python-software-properties
  - sudo apt-get update
  - sudo apt-get install -y -q innoextract wine python-software-properties
before_deploy:
  # This script runs when a release is tagged and creates the file to upload to the github release.
  # Below we create some variables, grab the version from the github release title, and replace the version text in the innosetup file.
  - dir1=$(pwd)
  - CONFIG_FILE="./installer/BeLL-Apps-Installer.iss"
  - TARGET_KEY="OutputBaseFilename"
  - BASE_NAME="-setup"
  - NEW_BELL_VER=`echo "$TRAVIS_TAG" | sed -nre 's/^[^0-9]*(([0-9]+\.)*[0-9]+).*/\1/p'`
  - REPLACEMENT_VALUE=$(printf 'BellApp-%s-setup' "$NEW_BELL_VER")

  - sed -i "s/BELL_VERSION/$NEW_BELL_VER/g" "$CONFIG_FILE"
  - sed -i "s/\($TARGET_KEY *= *\).*/\1$REPLACEMENT_VALUE/" $CONFIG_FILE
 

  - wine --version 
  - innoextract --version
  # Download inno installer, make sure we use a version that works with wine.
  - rm -rf /tmp/inno
  - mkdir /tmp/inno
  - cd /tmp/inno
  - wget -O is.exe http://files.jrsoftware.org/is/5/isetup-5.5.5.exe
  # Extract inno installer and copy it to wine drive C.
  - innoextract is.exe
  - mkdir -p ~/".wine/drive_c/inno"
  - cp -a app/* ~/".wine/drive_c/inno"
  - cd $dir1
  - ls ~/".wine/drive_c/inno"
  - ls ./installer/
  # Run our installer script through the inno setup compiler.
  - wine "C:\inno\ISCC.exe" $CONFIG_FILE
  - ls "./installer" -lah
  # Tell travis to export the file we just made.
  - export RELEASE_PKG_FILE=$(printf './installer/%s.exe' "$REPLACEMENT_VALUE")
  - #export RELEASE_PKG_FILE="./installer/curl.exe"
  - echo "deploying $RELEASE_PKG_FILE to GitHub releases"
deploy:
  provider: releases
  api_key:
    secure: "lU1rkysqVd/kBND129VD6wynoGK4c4v8JecgeBujeWQTXyZEgG0exhmZAPAafYZR5sqDDqbq+eFiCamj2mKYi4ybQYaS3fkI4xuFo2tR7p/oOqpGf2hZpkBYiFiI2Dm0aPXqH3w3Z+WmimYx/Qc0KsECDqFYPDYTKWMepXE8U1prllksavFiwW3YUVXXMKR4JOwtIMcqYq74l96LbpTGz4Vic4TARd34W+19sQasJ/QxK+5/RG3ToVohcv632OUPm60DQyOziuXEeTIpbGKLf7NAxxnuNjLbRnWTCgJBnfdt/K/HN/azvb+1PFBt1P1Nw2y8sDTRqZ5KGI5b4pNGvd8iOW9FZZgyCCiL/8y+yKse0Uds7wuH/pM8cLB4iEvT7GSoj0we5p/CsHGeSH2zxqYOY+dtO4k+eUihjVzEaTC9hhqitD7JgwbSLy0PRsBpyybuOr/GB0yZCDBiHvlnXxiIWgDllLsJUNx3iVT24b0yxhVX3RVfWiSfGx4FfbzWZ34SmpLns4h6oYFixOyRnBIvYYphvXwkNrV8FQJ6OPhNb+07EdJfu24bVuZX3VIw/HHbgrE+Cfzt0LEiKLiareTpy0z6+AMXaBgJ2XnX8vgAoeNGCTOaJ8uoqtwlNXQf5XuD6vpMp9Ix5HM40ytAHgeLjcUsqBgxMyiaIWgEDQY="
  # file_glob: true
  file: "${RELEASE_PKG_FILE}"
  # Don't delete the file we just made.
  skip_cleanup: true
  on:
tags: true
```


## What needs work

There is more work to be done:

- Right now the installation programs and scripts are located on a branch on `ole-vi/bell-apps` so that when a release is tagged it will start a travis build in the correct repository. It would be better if the installation stuff could be in a separate repo (the files are large) but still be triggered on a release from the bell apps repo.
- More testing, the installer has only been tried a few times.
- Better documentation, its hard to know what is difficult to understand when you are the one who made the project!
- Fix any issues relating to the inno installer [listed here](https://github.com/open-learning-exchange/Bell-Installer-for-Windows/issues).