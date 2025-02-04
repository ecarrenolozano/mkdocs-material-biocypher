# Install BioCypher

## System Requirements
[//]: # (TODO yaxi: Complete this section, add a table with Operative Systems tested, Python versions)
python >=3.10



## Prerequisites
1. Ensure that your Python version is 3.10 or higher. To check your current Python version, run the following command in
your terminal, Command Prompt, or PowerShell:
   ```bash
   python --version
   ```
2. Ensure you have `git` installed in your machine:
   ```bash
   git --version
   ```   

## Option 1. Use a pre-configured project with BioCypher

=== "local"
    !!! note "Note:"
        These are manual installation instructions. If you created the repository using the above GitHub template 
        functionality, you don't need to do the first two steps. Instead, just clone the repository you have created.


    ```bash
    # Clone this repository and rename to your project name.
    git clone https://github.com/biocypher/project-template.git
    mv project-template my-project
    cd my-project
   
    # Make the repository your own.
    rm -rf .git
    git init
    git add .
    git commit -m "Initial commit"
    # (you can add your remote repository here)
 
    # Install the dependencies using Poetry. 
    # Or feel free to use your own dependency management system. 
    # We provide a pyproject.toml to define dependencies.)
    poetry install
    
    #You are ready to go!
    poetry shell
    python create_knowledge_graph.py
    ```
=== "Docker"
    This repo also contains a docker compose workflow to create the example database using BioCypher and load it into a 
    dockerised Neo4j instance automatically. 
    
    Start up a single (detached) docker container with a Neo4j instance that contains the knowledge 
    graph built by BioCypher as the DB neo4j (the default DB),
    
    ```bash
    docker compose up -d 
    ```
    which you can connect to and browse at **localhost:7474**. 

    Authentication is deactivated by default and can be modified in the **docker_variables.env** file (in which case you 
    need to provide the .env file to the deploy stage of the docker-compose.yml).

    Regarding the BioCypher build procedure, the **biocypher_docker_config.yaml** file is used instead of the 
    **biocypher_config.yaml** (configured in scripts/build.sh). Everything else is the same as in the local setup. The first
    container (build) installs and runs the BioCypher pipeline, the second container (import) installs Neo4j and runs 
    the import, and the third container (deploy) deploys the Neo4j instance on localhost. The files are shared using a 
    Docker Volume. This three-stage setup strictly is not necessary for the mounting of a read-write instance of Neo4j, 
    but is required if the purpose is to provide a read-only instance (e.g. for a web app) that is updated regularly; 
    for an example, see the meta graph repository. The read-only setting is configured in the **docker-compose.yml** file 
    (NEO4J_dbms_databases_default__to__read__only: "false") and is deactivated by default.
    
    

## Option 2. Download a Package

=== "poetry(recommended)"
    !!! note "Note: About Poetry"
        Poetry is a tool for **dependency management** and **packaging** in Python. It allows you to declare the 
        libraries your project depends on and it will manage (install/update) them for you. Poetry offers a lockfile to 
        ensure repeatable installs, and can build your project for distribution. For information about the installation 
        process, you can consult [here](https://python-poetry.org/docs/#installation).

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




