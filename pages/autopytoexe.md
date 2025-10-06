# Create exe from Python code

### 1. Auto-py-to-exe

I will introduce a PyInstaller wrapper tool with GUI that I use for creating an executable file from smaller Python applications or scripts.

__* Issue__: I have an algorithm that is for a specific role. This algorithm is implemented in Python and works online, embedded in a Django web application. The issue is that the algorithm should run offline as a desktop application.

### 2. Creating exe from a main.py

#### How to start:

- Create a folder then set an environment using venv: 

  $ `python -m venv .venv`

  $ `.venv\scripts\activate`
- Install Auto-py-to-exe

    $ `pip install auto-py-to-exe`
- Copy the `requirements.txt` from the application's folder.

- Update the dependencies with auto-py-to-exe
    - add manually / `pip freeze > requirements.txt`


- Install all dependencies

    $ `pip install -r requirements.txt`
- Run Auto-py-to-exe

    $ `auto-py-to-exe`

#### Use the GUI:

1. Add __Script Location__ ( browse for a script [ main.py ] )
2. Select __One Directory__
3. Select __Console Based__ (I don't use windows/forms/panels in this case)
4. Add __Icon__ if you have one
5. __Additional Files__
    - Add the folders/files that are dependent on the application/project.
    - After the compilation you should delete the source code and other copyright content. The .pyc (compiled Python) files will be generated for the desktop application after execution. 
6. You can set the project name path & __Advanced/General Options/__ _*--name --contents-directory*_

#### Software info, rights

__Advanced/Windows specific options/__ _*--version-file*_

`version.txt`
```
# UTF-8
VSVersionInfo(
  ffi=FixedFileInfo(
    filevers=(1,0,0,0),
    prodvers=(1,0,0,0),
    mask=0x3f,
    flags=0x0,
    OS=0x4,
    fileType=0x1,
    subtype=0x0,
    date=(0,0)
    ),
  kids=[
    StringFileInfo(
      [
      StringTable(
        '040904B0',
        [StringStruct('CompanyName', 'Your Name'),
        StringStruct('FileDescription', 'What is it ...'),
        StringStruct('FileVersion', '1.0.0.0'),
        StringStruct('InternalName', 'my_app'),
        StringStruct('LegalCopyright', 'Firstname Lastname Copyright (C) 2025'),
        StringStruct('OriginalFilename', 'my_app.exe'),
        StringStruct('ProductName', 'My_App'),
        StringStruct('ProductVersion', '1.0.0.0')])
      ]), 
    VarFileInfo([VarStruct('Translation', [1033, 1200])])
  ]
)
```
The last step: 

CONVERT .PY TO .EXE