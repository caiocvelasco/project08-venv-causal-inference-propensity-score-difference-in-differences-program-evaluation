# Setting up Environment
* To check the Notebook for the Causal Project, go to its folder above (`causal_project`)

## Table of Contents

- [Project Structure](#project-structure)
- [Setup Instructions](#setup-instructions)
  - [Prerequisites](#prerequisites)
  - [Build and Run](#build-and-run)

## Project Structure

- **name_of_your_project_repo (project-root)/**
    - **.venv/**
    - **data/**
      - ...    
    - **img/**                   (A file with the images being used)
    - **.gitignore**             (A file to ignore what should not be pushed to the repo)
    - **.python-version**        (A file to control the python version being used in this project)
    - **requirements.txt**        (A file to install all needed dependencies)
    - **README.md**

## Setup Instructions

### Prerequisites

Make sure you have the following installed on your local development environment:

* [VSCode](https://code.visualstudio.com/)
  * Install `GitLens — Git supercharged` extension and Jupyter Notebook extensions.
* [Pyenv](https://github.com/pyenv-win/pyenv-win)
  * This will manage your Global Python Version (on your machine) and Local Python Versions (for your project).
  * Open Windows Powershell and follow the instructions from the Pyenv website, which are:
    * `Invoke-WebRequest -UseBasicParsing -Uri "https://raw.githubusercontent.com/pyenv-win/pyenv-win/master/pyenv-win/install-pyenv-win.ps1" -OutFile "./install-pyenv-win.ps1"; &"./install-pyenv-win.ps1"`
  * If there is an error, try this:
    * `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser`
  * Then, try the command above again
  * Open Git Bash:
    * pyenv --version (This will show if pyenv was installed successfully)
  * Create Pyenv Environment Variables in Windows
    * Windows Flag on your Keyboard + R:
      * sysdm.cpl -> Advanced Tab -> Environment Variables
      * Get the `pyenv-win` folder path. Usually, it is created here: `C:\Users\YOUR_USER\.pyenv\pyenv-win\`
      * Create 3 variables in the "User variables for YOUR_USER" part, and put the path above to each of them:
        * PYENV, PYENV_ROOT, PYENV_HOME
      * Double-click `Path` in the same part and click on New and add the paths for `bin` and for `shims`
        * `C:\Users\caiov\.pyenv\pyenv-win\bin`
        * `C:\Users\caiov\.pyenv\pyenv-win\shims`
        * Move both of them to the top (This will allow pyenv to have preference over the other python interpreters)
    * Check if it is working: Git Bash -> pyenv
  * Install Python Versions (This will install the python version that will be linked to different project with pyenv)
    * pyenv install 3.12.1
    * pyenv install 3.9.13
    * Check with: pyenv versions
  * Configure the Global Python version for your machine:
    * pyenv global 3.12.1
    * pyenv global (This will show the current global python version)
  * Configure the Local Python version for your project 
    * cd your_repo_folder
    * pyenv local 3.9.13 (This will create a `.python-version` file in your repo folder with the local version of your project)
    * Check with: python --version
* [.venv - Virtual Environment](https://docs.python.org/3/library/venv.html)
  * cd your_repo_folder:
    * cd "C:\...\causal-project"
  * python -m venv .venv           (This will create a virtual environment for the repo folder)
  * source .venv/Scripts/activate  (This will activate your virtual environment)
  * Install everything you need for your project from the `requirements.txt` file:
    * pip install --no-cache-dir -r requirements.txt  (This will install things inside your virtual environment)
    * maybe you need to update pip, this is important to avoid conflicts: python.exe -m pip install --upgrade pip
    * Check: pip list

  * [Git]
    * Make sure to inclue a .gitignore file with the following information:
      * .venv/         (to ignore the virtual environment stuff)
      * *.pyc          (to ignore python bytecode files)
      * data/          (to ignore the dataset)

### Build and Run

1. **Clone the repository:**

   ```bash
   git clone https://github.com/caiocvelasco/causal-project.git
   cd cd "C:\...\causal-project"

2. **Activate you Virtual Environment (.venv)**

* cd "C:\...\causal-project"
* source .venv/Scripts/activate                   (This will activate your environment)

3. **Causal Project**

* To check the project, go to its folder: `causal_project`