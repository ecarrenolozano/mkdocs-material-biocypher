# Install BioCypher

BioCypher is tested and supported on the following 64-bit systems

- Python 3.10-3.12
- Ubuntu 22.04 or later
- Windows 7 or later
- macOS 10.12.6 (Sierra) or later
- WSL via Windows 10


## Prerequisites
1. Ensure that your Python version is 3.10 or higher. To check your current Python version, run the following command in your terminal, Command Prompt, or PowerShell:
   ```bash
   python --version
   ```
2. Ensure you have `git` installed in your machine:
   ```bash
   git --version
   ```   

## Option 1. Use a pre-configured project with BioCypher

TO DO: Organize the information and include it here.

## Option 2. Download a Package

=== "poetry (recommended)"
    !!! note "Note: About Poetry"
        Poetry is a tool for **dependency management** and **packaging** in Python. It allows you to declare the libraries your project depends on and it will manage (install/update) them for you. Poetry offers a lockfile to ensure repeatable installs, and can build your project for distribution. For information about the installation process, you can consult [here](https://python-poetry.org/docs/#installation).

    ```bash
    # Create a new Poetry project, i.e. my-awesome-kg-project.
    poetry new <name-of-the-project>

    # Navigate into the recently created folder's project
    cd <name-of-the-project>
    
    # Install the BioCypher package with all the dependencies
    poetry add biocypher
    ```

=== "pip"

    !!! Note "Note: Virtual environment and best practices"
        To follow best practices in software engineering and prevent issues with your Python installation, we highly recommend installing packages in a separate virtual environment instead of directly in the base Python installation.

    1. **Create and activate** a virtual environment. Replace `<name-of-environment>` with the name of the environment you desire, i.e. `biocypher_env`    
        
        === "conda"

            ```bash
            # Create a conda environment with Python 3.10
            conda create --name <name-of-environment> python=3.10

            # Activate the new created environment
            conda activate <name-of-environment>
            ```
        
        === "venv"

            ```bash
            # Create a virtualenv environment
            python3 -m venv <name-of-environment>

            # Activate the new created environment
            source ./<name-of-environment>/bin/activate
            ```
     
    2. Install BioCypher package from `pip`. Type the following command to install BioCypher package. **Note:** do not forget to activate a virtual environment before do it.
   
        ```shell
        pip install biocypher        
        ```




