### [TOC](./TOC.md)
### [Back - Invoking transactions](./client.md)

# Misc other considerations
The section covers other small areas not mentioned in any other section

## Logging
Composer provided lots of logging through it's client api as well as the business network runtime, you should consider what logging will be required in your replacement. Another consideration is that composer provided th ability to turn logging on dynamically on the business network as well as the rest server. Dynamic logging on the contract is really useful but you will need to consider access control requirements to facilitate this.