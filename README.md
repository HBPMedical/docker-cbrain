# Docker instructions

* To build the containers:
```
./Docker/build.sh
```
The `build.sh` script has to be run from the base CBRAIN installation directory, not from `Docker`.

* To start a CBRAIN portal:
```
cd Docker
source .env
docker-compose up
```
Be careful to write down the CBRAIN admin password the first time you run `docker-compose up`.

The CBRAIN portal is then available at http://localhost:3000. It has a data provider, a bourreau, and the Diagnostics tool configured.

* Files and directories
  * `Dockerfiles`: Dockerfiles for the base CBRAIN, portal and bourreau images.
  * `entry_points`: Bash scripts used as container entry points.
  * `init_portal`: Ruby scripts used to initialize the database with a data provider, a bourreau and a tool.
  * `templates`: Templates for portal and bourreau configuration files, instantiated in the entry point scripts.
  * `volumes`: Directory where the mounted persistent volumes are created.

* Startup process

The following containers are started (see configuration in `docker-compose.yml`):
1. MariaDB database
2. CBRAIN portal
3. CBRAIN bourreau

These containers mount volumes that are all located in `volumes`.

The entry point of the portal and bourreau are the bootstrap scripts
(`{portal,bourreau}_bootstrap.sh`). These scripts are run as user
`root` to adjust the owners and permissions of the mounted
volumes. They launch `{portal,bourreau}.sh` as user `cbrain`.

The first time the portal starts, it:
* Loads the database schema
* Seeds the database
* Sanity checks the database
* Runs all the scripts in `init_portal` in the Rails console of the portal:
  * `create_dp.rb`: creates a local data provider, that persists files in `volumes/data_provider`.
  * `create_bourreau.rb`: adds a bourreau to the database, that corresponds to the bourreau container.
  * `create_diagnostics_tool_config.rb`: loads the Tools and creates a ToolConfig for Diagnostics on the previously-created bourreau.

Everytime the portal starts, it:
* Waits for the database to be available.
* Installs the plugins in `volumes/portal/plugins`.
* Launches the BrainPortal application.

Everytime the bourreau starts, it:
* Installs the plugins in `volumes/bourreau/plugins`.
* Waits for the portal to be started.
* Copy the portal's public key in its `authorized_keys` file.
* Starts `sshd`.    
