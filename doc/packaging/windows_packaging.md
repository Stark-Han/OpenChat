<p align="left">
  <a href="../../README.en.md">Back</a> 

## Windows application packaging process and configuration method
To build an application installation package in the Windows environment, you need to rely on the following environments and applications:
- **Python environment**：Python3.10.11/3.10.12  
- **Application packaging tools**：Inno Setup Compiler
### 1 Install the Python environment
Download the Python installation package:[Visit the official Python website](https://www.python.org/),DownloadPython3.10.11
### 2 Build a application packaging virtual environment
Activate the previously deployed Python environment
```shell
pip install virtualenv
virtualenv venv --python=python3.10.11
```
### 3 Install project dependencies in a venv environment
Go to Venv to activate this environment and install the dependencies according to the requirements.txt provided by this project
```shell
pip install -r requirements.txt
```
Please modify some of the code in the init.py in the pypandoc package
```python
stdout.decode('utf-8') ---> stdout
```
Please modify some of the code in the pytesseract.py of the pytesseract package
```python
tesseract_cmd = r'.\_internal\Tesseract_OCR\tesseract.exe'
```
Please put the nltk_data provided by the project in the ./venv/lib of the virtual environment
### 4 Build the executable exe
Use pyinstaller to generate an openchat.spec file in the root directory of your project
```shell
pyinstaller -D openchat.py
```
Run openchat.spec to generate a build folder and a dist folder in the root directory of the project, where the dist folder is the content related to the final generation of the application installation package
```shell
pyinstaller opeanchat.spec
```
In the project, you can see the dependency package _internal, front-end asset, and openchat.exe
![](../../data/images/packaging/windows.png)
Please use powershell to execute openchat.exe, usually the console will print an error of missing dependency packages, please add the missing dependency packages to the _internal in the virtual environment ./venv/lib/site-packages;
In addition to the missing dependencies, put pkg, Tesseract_OCR, and poppler-0.89.0 into the _internal;
Please add the URLLlib provided in the project to the _internal
### 5 Build an application installation package
Download from the official website[Inno Setup Compiler](https://jrsoftware.org/isdl.php),This project provides an application build script openchatsetup.iss,Please fill in the name of the personal packaging path, and modify other parameters according to your personal needs, the build process depends on the configuration of the computer, usually kept between 10-15 minutes
```pascal
; Script generated by the Inno Setup Script Wizard.
; SEE THE DOCUMENTATION FOR DETAILS ON CREATING INNO SETUP SCRIPT FILES!

#define MyAppName "OpenChat"
#define MyAppVersion "1.0"
#define MyAppPublisher "xxx"
#define MyAppURL "https://www.xxx.com/"
#define MyAppExeName "openchat.exe"
#define MyAppAssocName MyAppName + " File"
#define MyAppAssocExt ".myp"
#define MyAppAssocKey StringChange(MyAppAssocName, " ", "") + MyAppAssocExt

[Setup]
; NOTE: The value of AppId uniquely identifies this application. Do not use the same AppId value in installers for other applications.
; (To generate a new GUID, click Tools | Generate GUID inside the IDE.)
AppId={{D8996EC5-54ED-4916-B820-E79C648E59A8}
AppName={#MyAppName}
AppVersion={#MyAppVersion}
;AppVerName={#MyAppName} {#MyAppVersion}
AppPublisher={#MyAppPublisher}
AppPublisherURL={#MyAppURL}
AppSupportURL={#MyAppURL}
AppUpdatesURL={#MyAppURL}
DefaultDirName={autopf}\{#MyAppName}
; "ArchitecturesAllowed=x64compatible" specifies that Setup cannot run
; on anything but x64 and Windows 11 on Arm.
ArchitecturesAllowed=x64compatible
; "ArchitecturesInstallIn64BitMode=x64compatible" requests that the
; install be done in "64-bit mode" on x64 or Windows 11 on Arm,
; meaning it should use the native 64-bit Program Files directory and
; the 64-bit view of the registry.
ArchitecturesInstallIn64BitMode=x64compatible
ChangesAssociations=yes
DisableProgramGroupPage=yes
LicenseFile=/xxx/xxx/xxx // LicenseFile
; Uncomment the following line to run in non administrative install mode (install for current user only.)
;PrivilegesRequired=lowest
OutputDir=/xxx/xxx/xxx // The path of the output file after the exe is packaged
OutputBaseFilename=OpenChatSetup
SetupIconFile=/xxx/xxx/xxx // Install icon
Compression=lzma
SolidCompression=yes
WizardStyle=modern

[Code]
procedure CurStepChanged(CurStep: TSetupStep);
var
  uninspath, uninsname, NewUninsName, MyAppName: string;
  sUnInstPath: String;
  sUnInstallExe: String;
  iResultCode: Integer;
  OldFilePath, NewFilePath: string;
  FilePath: string;
begin

  if CurStep = ssInstall then
  begin
    sUnInstPath := ExpandConstant('SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\');
    sUnInstallExe := '';
    if RegQueryStringValue(HKEY_LOCAL_MACHINE, sUnInstPath, '{#MyAppName}_unisl', sUnInstallExe) then
    begin
      sUnInstallExe := RemoveQuotes(sUnInstallExe);
      Exec(sUnInstallExe, '/SILENT /NORESTART', '', SW_HIDE, ewWaitUntilTerminated, iResultCode)
      // Delete the old version of the data table
      FilePath := ExpandConstant('C:\Users\{%username|DefaultValue}\.openchat\openchat_old.db');
      if FileExists(FilePath) then
      begin
        DeleteFile(FilePath);
      end
    end
   end;


  if CurStep = ssDone then
  begin
    NewUninsName := 'OpenChatUninstall';
    MyAppName := '{#MyAppName}';
    
    // Rename the uninstall file
    uninspath := ExtractFilePath(ExpandConstant('{uninstallexe}'));
    uninsname := Copy(ExtractFileName(ExpandConstant('{uninstallexe}')), 1, 8);
    RenameFile(uninspath + uninsname + '.exe', uninspath + NewUninsName + '.exe');
    RenameFile(uninspath + uninsname + '.dat', uninspath + NewUninsName + '.dat');

    // Rename the db
    OldFilePath := ExpandConstant('C:\Users\{%username|DefaultValue}\.openchat\openchat.db');
    NewFilePath := ExpandConstant('C:\Users\{%username|DefaultValue}\.openchat\openchat_old.db');
    if FileExists(OldFilePath) then
    begin
      RenameFile(OldFilePath, NewFilePath);
    end
  end;
end;


[Languages]
Name: "english"; MessagesFile: "compiler:Default.isl"

[Tasks]
Name: "desktopicon"; Description: "{cm:CreateDesktopIcon}"; GroupDescription: "{cm:AdditionalIcons}"; Flags: unchecked

[Files]
// In this case, please fill in the full path after packaging with pyinstaller
Source: "xxx\dist\yuanchat\{#MyAppExeName}"; DestDir: "{app}"; Flags: ignoreversion
Source: "xxx\dist\yuanchat\*"; DestDir: "{app}"; Flags: ignoreversion recursesubdirs createallsubdirs
; NOTE: Don't use "Flags: ignoreversion" on any shared system files

[Registry]
Root: HKLM; Subkey: "Software\Microsoft\Windows\CurrentVersion\Uninstall\"; ValueType: string; ValueName: "{#MyAppName}_unisl"; ValueData: "{app}\OpenChatUninstall.exe";Flags: uninsdeletevalue

[Icons]
Name: "{autoprograms}\{#MyAppName}"; Filename: "{app}\{#MyAppExeName}"
Name: "{autodesktop}\{#MyAppName}"; Filename: "{app}\{#MyAppExeName}"; Tasks: desktopicon

[Run]
Filename: "{app}\{#MyAppExeName}"; Description: "{cm:LaunchProgram,{#StringChange(MyAppName, '&', '&&')}}"; Flags: nowait postinstall skipifsilent


```