# Hyperledger Composer Operations Guide
- [Introduction](./introduction.md)
- [Hyperledger Composer Security Model](./idsandparts.md)
- [Connection Profiles](./connectionprofiles.md)
- [Business network cards](./busnetcards.md)
- [Network Admin participant type](./networkadmin.md)
- [Fabric CA - TBD](./fabric-ca.md)
- [Deploying business networks](./deploy.md)
- [Upgrading business networks](./upgrade.md)
- [Managing identities and participants](./managingids.md)
- [ACLs for operations](./acls.md)
- [Cloud Wallets - TBD](./cloud-wallets.md)
- [Hyperledger Composer docker images - TBD](./tbd.md)
- [Hyperledger Composer Rest Server - TBD](./tbd.md)
- [Client connectivity and network reliability handling - TBD](./tbd.md)
- [Hardware Security Module (HSM) support](./tbd.md)
- [using an alternative certificate provider - TBD](./tbd.md)
- [startBusinessNetwork transaction - TBD](./tbd.md)


- Things that need to be included
  - Need to note about identity activation

  - client version mismatch with runtime version
  - version migration ????
  - commands that create cards for you and the issues there
  - CORE_CHAINCODE_EXECUTETIMEOUT
  - CORE_VM_DOCKER_ATTACHSTDOUT
  - CORE_CHAINCODE_LOGGING
  - CORE_PEER_LOGGING
  - orderer logging
  - ca server logging
  - include fabric-ca-client in managing identities and participants
  - upgrade should include endorsement policies
  - cloud wallets
    - wallet to hold sensitive info such as cards, connector info eg fabric keystore for keys & certs
    - can share a wallet through filesystem sharing, remote server
    - danger with fabric-ca admin if you store card with a secret, if 2 people use it at the same time, 2 enrollments can occur
  - fabric-ca-client with tls issues



