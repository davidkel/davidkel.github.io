# Hyperledger Composer Operations Guide (TBD - No Content yet)
- [Introduction](./introduction.md)
- [Using Online Playground](./onlineplayground.md)
- [Setting up a Local development environment](./localdev.md)
- [Creating Models](./models.md)
- [Writing TP Functions](./tpfunctions.md)
- [Writing Queries](./queries.md)
- [Writing ACLs](./acls.md)
- [Writing Unit tests for business networks](./unittests.md)
- [Writing Client Applications](./clientapps.md)
- [Writing Angular Applications](./angularapps.md)
- [Utilising a standalone runtime server](./simulator.md)
- [Utilising a real Fabric](./realfabric.md)
- [REST API](./restapi.md)
- [Testing events with Node-Red](./nodered.md)
- [Using Fabric API in TP Functions](./fabricapi.md)
- [External models](./externalmodels.md)

## Things to include
- how to set up a development environment on
  - Mac
  - Linux
  - Windows
  - Any: https://github.com/jt-nti/composer-devenv which does an all in one, but need to edit files in the vagrant directory.
- how to use fabric in developer mode
- use web browser mode of playground as much as possible
- use a fabric simulator, which I have developed.
- CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=tutorial-network_default
  Here "tutorial-network_default" is the network name. You can find your network name by using "docker network ls" command
- how to debug ACLs or run a simulation to get results
- how to debug queries
- script to package, install, upgrade

