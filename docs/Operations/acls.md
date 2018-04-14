### [TOC](./TOC.md)


#ACls

ACL's are not only used to provide access control for business interactions, they also provided access control for operational aspects as well. Unfortunately at this time there is only a single ACL file so no easy way to seperate the needs and requirements of administrators to that of the business network itself and will be the role of the business network developer to develop the administrator controls as well.

For the most part, it's likely for operational aspects that the resources to be under control with be those in `org.hyperledger.composer.system.cto`
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
| AssetRegistry | An Registry registry, will create an AssetRegistry registry and it will hold a list of all the Asset Types in both system and Business network|
| ParticipantRegistry | A Participant Registry registry, will create a ParticipantRegistry registry and it will hold a list of all the Participant Types in both system and Business network|
| TransactionRegistry | A TransactionRegistry registry. will create a TransactionRegistry regsitry and it will hold a list of all the Transaction Types in both system and Business network|
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
The following sections provide some guidance on the rules, however you should consider that as the number of ACL rules grows, performance will be reduced. So you might want to consider using more generic types rather than specific types. Another approach could be to define `DENY` rules followed by a rule that allows everything in the system namespace. ACL rules a processed top to botton so the deny rules will be actioned first.
### Rules that everyone should have
There are going to be some rules that everyone (including participants transacting on the business network) need to be able to do anything on the business network.

| resource | operation | description        |
| -------- | --------- | ------------       |
| Network | READ | Everyone must have read access to the business network, there may be exceptions where this is not the case but this is a recommendation |
| AssetRegistry#HistorianRecord | READ | Everyone who will manipulate an asset, participant or submit a transaction will need to be able to read the Historian Record structure in order to be able to create one |
| HistorianRecord | CREATE | Everyone who will manipulate an asset, participant or submit a transaction will need to be able to create a Historian Record |

### Rules to permit the creation/reading/updating/deleting of any participant

| resource | operation | description        |
| ParticipantRegistry | READ | Need this to see all the participant registries |
| TransactionRegistry#AddParticipant | READ | Allows you to read the registry for system transactions, you need this to be able to get the system transaction you want to create |
| AddParticipant | CREATE, READ | Allows you to create the AddParticipant transaction object, read access so you can see it in historian |
| Participant | CREATE, READ | Allows you to create any (system or otherwise) type of Participant Type, and then be able to see that it was created |
| TransactionRegistry#UpdateParticipant | READ | |
| UpdateParticipant | CREATE, READ | Allows you to create and read later in historian the UpdateParticipant transaction object |
| Participant | CREATE, READ, UPDATE, DELETE | Allows you to create any (system or otherwise) Participant Type, and then be able to see that it was created and then update the participant |


### Rules to be able to associate identities with a partipant
It's assumed that you will have already granted some sort of read access to the Participant type.

| AssetRegistry#Identity | READ | Ability to view the identity registry |
| Identity | CREATE, READ | Read is required to read the list of identities, CREATE is required to create a new Identity asset |
| TransactionRegistry#IssueIdentity | READ | Need to be able to read the system transaction IssueIdentity |
| IssueIdentity | CREATE, READ | Need to be able to create the system transaction IssueIdentity, read so that can view historian record |


### example ACL file putting it all together
This example assumes that you are using the NetworkAdmin participant type for all Business Network administration. ie, they can create all types of participant, they an issue or bind identities to a participant. They can revoke identities, finally the ability to only see those specifics in the Historian. This example shows all the rules required but may not be the most efficient way. But it ensures the network admin can only do what they are allowed to do. 

(TBD is there a better rule set to achieve the same thing ?)

```
AllCanReadBusinessNetwork {
    description: "Must be able to read the business network to be able to do anything on a business network connection as it gets a business network"
    participant: "ANY"
    operation: READ
    resource: "org.hyperledger.composer.system.Network"
    action: ALLOW
}

AllCanReadHistorianRecordStructure {
    description: "Everyone who will manipulate an asset, participant or submit a transaction will need to be able to read the Historian Asset definition. "
    participant: "ANY"
    operation: READ
    resource: "org.hyperledger.composer.system.AssetRegistry#org.hyperledger.composer.system.HistorianRecord"
    action: ALLOW
}

AllCanCreateHistorianRecord {
    description: "Everyone who will manipulate an asset, participant or submit a transaction will need to be able to create a Historian Record"
    participant: "ANY"
    operation: CREATE
    resource: "org.hyperledger.composer.system.HistorianRecord"
    action: ALLOW
}

rule NetworkAdminParticipantRegistryAccess {
    description: "Network admin can see all the available Participant Registries"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: READ
    resource: "org.hyperledger.composer.system.ParticipantRegistry"
    action: ALLOW
}

rule NetworkAdminAddParticipant {
    description: "Network admin can read the AddParticipant Transaction structure"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: READ
    resource: "org.hyperledger.composer.system.TransactionRegistry#org.hyperledger.composer.system.AddParticipant"
    action: ALLOW
}

rule NetworkAdminAddParticipant2 {
    description: "Allows Network admin to create the AddParticipant transaction object, read access so you can see it in historian"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: CREATE, READ
    resource: "org.hyperledger.composer.system.AddParticipant"
    action: ALLOW
}

rule NetworkAdminManageParticipant {
    description: "Allows Network admin to create, update or delete any (system or otherwise) type of Participant Type, and then be able to see that it was created"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: ALL
    resource: "org.hyperledger.composer.system.Participant"
    action: ALLOW
}

rule NetworkAdminUpdateParticipant {
    description: "Allows network admin to read the Update Participant system transaction"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: READ
    resource: "org.hyperledger.composer.system.TransactionRegistry#org.hyperledger.composer.system.UpdateParticipant"
    action: ALLOW
}

rule NetworkAdminUpdateParticipant2 {
    description: "Allows you to create and read later in historian the UpdateParticipant transaction object"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: CREATE, READ
    resource: "org.hyperledger.composer.system.UpdateParticipant"
    action: ALLOW
}

// add Delete Participant
// org.hyperledger.composer.system.TransactionRegistry#org.hyperledger.composer.system.RemoveParticipant

rule NetworkAdminIdentityRegistryAccess {
    description: "Ability to view the identity registry"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: READ
    resource: "org.hyperledger.composer.system.AssetRegistry#org.hyperledger.composer.system.Identity"
    action: ALLOW
}

rule NetworkAdminIdentityAssetsAccess {
    description: "Read is required to read the list of identities, CREATE is required to create a new identity"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: CREATE, READ
    resource: "org.hyperledger.composer.system.Identity"
    action: ALLOW
}

Ability to Issue an Identity
----------------------------
rule NetworkAdminIssueIdentityAccess {
    description: "Need to be able to read the system transaction IssueIdentity"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: READ
    resource: "org.hyperledger.composer.system.TransactionRegistry#org.hyperledger.composer.system.IssueIdentity"
    action: ALLOW
}

rule NetworkAdminCreateIssueIdentityTx {
    description: "Need to be able to create the system transaction IssueIdentity, read so that can view historian record"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: CREATE, READ
    resource: "org.hyperledger.composer.system.IssueIdentity"
    action: ALLOW
}


// need to add bind and revoke
// Need to add historian access.
```
