### [TOC](./TOC.md)
### [Back - Query Language](./querylang.md)

# ACLs

There is no equivalent to ACLs in fabric. Acls controlled what participants could do what with various resources, which included Composer system resources as well as the applications defined data resources. You will need to create your own concept on how to perform access control. When an identity was used to access fabric, this was mapped to a composer participant. You can get the identity used to invoke the transaction using the `getCreator` api or more likely you should using the `client identity` api which extracts the public certificate and provides easy access to information about this certificate. You can also make use of attribute based access control by putting attributes into the certificate you issue to users and then using the client identity package to access those attributes to make access control based decisions. 

TODO: Update the reference to client identity.

### [Next - Historian](./historian.md)