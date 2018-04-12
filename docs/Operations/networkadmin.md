### [TOC](./TOC.md)
### [Back - Business Network Cards](./busnetcards.md)

# NetworkAdmin Participant type
Let's dispell the myth right now. There is **NO** inbuilt Composer administrator for a business network. What about the `admin` id I hear you say and the participant that I see called `org.hyperledger.composer.system.NetworkAdmin#admin` surely they are the administrator identity and participant. 
Sorry to say they are not and there is no inbuilt superpowers for an identity with a name of `admin` (remember the name is taken from the certificate's Common Name field) or a participant `org.hyperledger.composer.system.NetworkAdmin#admin` or it's participant type `org.hyperledger.composer.system.NetworkAdmin`.

## So what is `NetworkAdmin` and why does it appear to be the administrator type ?

NetworkAdmin is just another participant type, identical to any other participant type you might define when you model your solution in Composer, it's just that it's built into the Composer system

(From the system.cto source file)
```
participant NetworkAdmin identified by participantId {
    o String participantId
}
```

Why is it there you ask ? Well as you recall in order to do anything on a business network you must have a participant and that participant is mapped to an identity. It would be a bit of a problem if you deployed a business network that had no participants created and no identities mapped to it, you wouldn't be able to use the business network at all. We'll discuss this more in the section on deploying business networks but basically it is a well known participant type Composer can use at deploy time to ensure your business network has at least one participant mapped to an identity to ensure you have access.

## Why does NetworkAdmin participant types appear to have superpowers ?
That is all down to the ACLs you provide in your business network. The samples, generators and playground create a default set of ACL's that you usually include in your business network which look like this 

```
rule NetworkAdminUser {
    description: "Grant business network administrators full access to user resources"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: ALL
    resource: "**"
    action: ALLOW
}

rule NetworkAdminSystem {
    description: "Grant business network administrators full access to system resources"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: ALL
    resource: "org.hyperledger.composer.system.**"
    action: ALLOW
}
```
Ideally you wouldn't ship these as is as part of your business network, you would restrict the role of the NetworkAdmin participant type to only be able to perform actions specific to the role you want them to perform. 
The first ACL gives access to obsolutely everything, so if you leave that one in you don't need the second one. The second ACL gives the NetworkAdmin access only to the resources defined in the system model. We will discuss practical ACLs for different roles later in this book.

### [Next - Deploying Business Networks](./tbd.md)
