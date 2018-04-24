### [TOC](./TOC.md)

### [Back - Managing identities and participants](./managingids.md)

#ACLs
ACL's are not only used to provide access control for business interactions, they also provided access control for operational aspects as well. Unfortunately at this time there is only a single ACL file so no easy way to seperate the needs and requirements of administrators to that of the business network itself and will be the role of the business network developer to develop the administrator controls as well.

Some useful things to know about ACL and ACL files

- if no ACL file is provided, then every participant can do anything, there are no controls in place.
- The higher the number of ACL rules the more this will impact performance as both submitted transactions perform checks as well as query requests against all resources touched.

## resources in system.cto
For the most part, it's likely for operational aspects that the resources to be under control with be those in `org.hyperledger.composer.system.cto` file found in `composer-common/lib/system`. Here is a summary of those resources and meaning. For brevity the `org.hyperledger.composer.system` namespace have been removed.


![Diagram](./hierarchy.png)

| resource | comments |
| -------- | -------- |
| Asset (abstract) | Any type of asset|
| Participant (abstract)| Any type of participant|
| Transaction (abstract)| Any type of transaction|
| Event (abstract)| Any type of Event |
| Registry (abstract)| Any type of registry |
| AssetRegistry | An Asset Registry registry, will create an AssetRegistry registry and it will hold a list of all the Asset Registries in both system and Business network|
| ParticipantRegistry | A Participant Registry registry, will create a ParticipantRegistry registry and it will hold a list of all the Participant Registries in both system and Business network|
| TransactionRegistry | A TransactionRegistry registry. will create a TransactionRegistry registry and it will hold a list of all the Transaction Registries in both system and Business network|
| Network | type of Asset that represents a business network will also have a Network registry to hold instances |
| NetworkAdmin | type of Participant, will also have a NetworkAdmin registry to hold instances|
| HistorianRecord | type of Asset , will also have a HistorianRecord registry to hold instances|
| RegistryTransaction (abstract)| type of Transaction class |
| AssetTransaction (abstract)| type of Registry Transaction |
| ParticipantTransaction (abstract)| type of Registry Transaction |
| AddAsset | type of transaction, will also have a transaction registry to hold instances of it |
| UpdateAsset | type of transaction, will also have a transaction registry to hold instances of it |
| RemoveAsset | type of transaction, will also have a transaction registry to hold instances of it |
| AddParticipant | type of transaction, will also have a transaction registry to hold instances of it |
| UpdateParticipant | type of transaction, will also have a transaction registry to hold instances of it |
| RemoveParticipant | type of transaction, will also have a transaction registry to hold instances of it |
| Identity | type of Asset will have a registry to hold instances of |
| IssueIdentity | type of transaction, will also have a transaction registry to hold instances of it |
| BindIdentity | type of transaction, will also have a transaction registry to hold instances of it |
| ActivateCurrentIdentity | type of transaction, will also have a transaction registry to hold instances of it |
| RevokeIdentity | type of transaction, will also have a transaction registry to hold instances of it |
| StartBusinessNetwork | type of transaction, will also have a transaction registry to hold instances of it |
| ResetBusinessNetwork | type of transaction, will also have a transaction registry to hold instances of it |
| SetLogLevel | type of transaction, will also have a transaction registry to hold instances of it |

## Rules
The following sections provide some guidance on the rules, however you should consider that as the number of ACL rules grows, performance will be reduced. So you might want to consider using more generic types rather than specific types. Another approach could be to define `DENY` rules followed by a rule that allows everything in the system namespace. ACL rules are processed top to botton so the deny rules will be actioned first.
### Rules that everyone should have
There are going to be some rules that everyone (including participants transacting on the business network) need to be able to do anything on the business network.

| resource | operation | description        |
| -------- | --------- | ------------       |
| Network | READ | Everyone must have read access to the business network, there may be exceptions where this is not the case but this is a recommendation |
| AssetRegistry#HistorianRecord | READ | Everyone who will manipulate an asset, participant or submit a transaction will need to be able to read the Historian Record structure in order to be able to create one |
| HistorianRecord | CREATE | Everyone who will manipulate an asset, participant or submit a transaction will need to be able to create a Historian Record |

### Rules to permit the creation/reading/updating/deleting of any participant

| resource | operation | description        |
| -------- | --------- | ------------       |
| ParticipantRegistry | READ | Need this to see all the participant registries |
| TransactionRegistry#AddParticipant | READ | Required to be able to access the AddParticipant Transaction Registry |
| AddParticipant | ALL | Allows you to create/update/delete/read the AddParticipant transaction object, read access so you can see it in historian |
| Participant | CREATE, READ | Allows you to create any (system or otherwise) type of Participant Type, and then be able to see that it was created |
| TransactionRegistry#UpdateParticipant | READ | Required to be able to access the UpdateParticipant Transaction Registry|
| UpdateParticipant | CREATE, READ | Allows you to create and read later in historian the UpdateParticipant transaction object |
| TransactionRegistry#RemoveParticipant | READ | Required to be able to access the RemoveParticipant Transaction Registry|
| RemoveParticipant | CREATE, READ | Allows you to create and read later in historian the RemoveParticipant transaction object |

### Rules to be able to associate identities with a partipant
It's assumed that you will have already granted some sort of read access to the Participant type.

| resource | operation | description        |
| -------- | --------- | ------------       |
| AssetRegistry#Identity | READ | Ability to view the identity registry |
| Identity | CREATE, READ | Read is required to read the list of identities, CREATE is required to create a new Identity asset |
| TransactionRegistry#IssueIdentity | READ | Need to be able to access the transaction registry IssueIdentity |
| IssueIdentity | CREATE, READ | Need to be able to create the system transaction IssueIdentity, read so that can view historian record |

(repeat the IssueIdentity definitions, for BindIdentity, RevokeIdentity, ActivateCurrentIdentity, StartBusinessNetwork)

### example ACL file putting it all together
Here's an example of an ACL that provides a network administrator with the following capabilities
- Able to create/update/delete any participant type
- Able to perform identity management
- Able to view all the historian records, but can only see the contents of records related to network admin activities

Also considered here is how to achieve what is needed with a few rules as possible, without using conditions.

it assumes that you are using the NetworkAdmin participant type for all Business Network administration.

it's important to note that inheritance only works on the type and not the specific instance, for example this would not work
resource: "org.hyperledger.composer.system.TransactionRegistry#org.hyperledger.composer.system.ParticipantTransaction"

to refer to AddParticipant, UpdateParticipant, RemoveParticipant transactions stored in the org.hyperledger.composer.system.TransactionRegistry

but you could replace `org.hyperledger.composer.system.TransactionRegistry` with `org.hyperledger.composer.system.Registry` and that would work.

```
rule AllCanReadBusinessNetwork {
    description: "Must be able to read the business network to be able to do anything on a business network connection as it gets a business network"
    participant: "ANY"
    operation: READ
    resource: "org.hyperledger.composer.system.Network"
    action: ALLOW
}

rule AllCanReadHistorianRegistry {
    description: "Everyone who will manipulate an asset, participant or submit a transaction will need to be able to read the Historian Transaction Registry"
    participant: "ANY"
    operation: READ
    resource: "org.hyperledger.composer.system.AssetRegistry#org.hyperledger.composer.system.HistorianRecord"
    action: ALLOW
}

rule AllCanCreateAndReadHistorianRecord {
    description: "Everyone who will manipulate an asset, participant or submit a transaction will need to be able to create a Historian Record, Let everyone be able to read it as well"
    participant: "ANY"
    operation: CREATE, READ
    resource: "org.hyperledger.composer.system.HistorianRecord"
    action: ALLOW
}

rule NetworkAdminIdentityRegistryAccess {
    description: "Network admin can access the identity asset registry"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: READ
    resource: "org.hyperledger.composer.system.AssetRegistry#org.hyperledger.composer.system.Identity"
    action: ALLOW
}

rule NetworkAdminNoAssetRegistryAccess {
    description: "Deny the Network Admin access to any of the Asset Registries"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: READ
    resource: "org.hyperledger.composer.system.AssetRegistry"
    action: DENY
}

rule NetworkAdminRegistryAccess {
    description: "Network admin can see all the available Participant Registries, and Transaction Registries, but not asset registries, except identity, as dictated above"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: READ
    resource: "org.hyperledger.composer.system.Registry"
    action: ALLOW
}

rule NetworkAdminManageParticipant {
    description: "Allows Network admin to create, update or delete any type of Participant Type, and then be able to see read the Participants"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: ALL
    resource: "org.hyperledger.composer.system.Participant"
    action: ALLOW
}

rule NetworkAdminIdentityAccess {
    description: "Allow a Network Admin to create an identity and read created identities"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: CREATE, READ
    resource: "org.hyperledger.composer.system.Identity"
    action: ALLOW
}

rule NetworkAdminIssueIdentityAccess {
    description: "Allows Network admin to create an instance of an issue identity transaction, read access so the details can be seen"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: CREATE, READ
    resource: "org.hyperledger.composer.system.IssueIdentity"
    action: ALLOW
}

rule NetworkAdminBindIdentityAccess {
    description: "Allows Network admin to create any instance of an bind identity transaction, read access so the details can be seen"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: CREATE, READ
    resource: "org.hyperledger.composer.system.BindIdentity"
    action: ALLOW
}

rule NetworkAdminRevokeIdentityAccess {
    description: "Allows Network admin to create any instance of an bind identity transaction, read access so the details can be seen"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: CREATE, READ
    resource: "org.hyperledger.composer.system.RevokeIdentity"
    action: ALLOW
}

rule NetworkAdminStartBusinessNetworkAccess {
    description: "Allows Network admin to create any instance of an bind identity transaction, read access so the details can be seen"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: CREATE, READ
    resource: "org.hyperledger.composer.system.StartBusinessNetwork"
    action: ALLOW
}

rule NetworkAdminParticipantAccess {
    description: "Allows Network admin to create an add/update/remove participant, read access so the details can be seen"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: CREATE, READ
    resource: "org.hyperledger.composer.system.ParticipantTransaction"
    action: ALLOW
}
```

Ideally it would be good if IssueIdentity, BindIdentity, RevokeIdentity all inherited from an IdentityTransaction, then we could collapse 3 rules into 1 and also include ActivateCurrentIdentity as well, but unfortunately that isn't how the system model is designed currently.

### [Next - Cloud Wallets](./cloud-wallets.md)
