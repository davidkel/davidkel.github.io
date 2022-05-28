# Stuff todo

## FabricDeveloper
- node client
- Go client

## vscode
- node client
- GO client and IBP V2

## New section on sandbox tools
- cdsunpacker
- certificate tools for composer
- tool to restore access to locked out business network

## fabric book

- node client HA
- node chaincode (note about pagination queries can only be used in read only txns)
The pagination APIs are for use in read-only transactions only, the query results are intended to support client paging requirements. For transactions that need to read and write, use the non-paginated chaincode query APIs. Within chaincode you can iterate through result sets to your desired depth.
- new lifecycle
- external chaincode
- contract api
  - namespaced and how to call
  - does inheritance work
  - replacing the serializer
  - built in txn

### Submit

- I select the peers (How do I find out the peers ?)
- I select the orgs (you select the peers) (How do I find out the orgs)
- Satisfy the endorsement policy
- All (but you select the peers)


### Query

- I Select the peer
- Allow for multiple peers and compare
- Select a peer in my org
- Round robin

### Events

Commit, Chaincode, block

strategies for commit

strategies for chaincode

strategies for block


Submit can't return private data

new sdk behaviour changes things again
1. sending transient data is cautious so won't work with implicit collections with SBEs that require 2 endorsements for example
