# Go SDK Further experiences
This page recounts my experience with the Go SDK working with fabric 1.4. 
When I started looking at this I wasn't sure if the latest commit to master would work with fabric 1.4 as some of the commits look as though they had moved toward the 2.0.0-alpha release. I tried to use the alpha5 tag on master but this would not compile so I started with alpha4 tag as I would assume that it would perhaps be more compatible with fabric 1.4. My original investigation showed there that the BCCSP section had to be provided. However this was made optional later. Also alpha4 used godep as it's dependency handler. Moving to the latest (june 4th) commit on master I have found the following
1. BCCSP is optional now (but that does have it's own implications)
2. go modules can be used
3. It appears to still work with 1.4 although I've not tried anything complex or any of the operational apis such as install/instantiate
4. I also note that there hasn't been any support added for the new lifecyle api coming on 2.0 also if there are any other 2.0 features requiring sdk support/changes I'm not sure these will be added
5. A couple of fixes have been added since the jun 4th release but on the whole there currently isn't much activity. I have not looked at the JIRAs to see what is on this list of features or defects.
6. It is possible to replace the persistence mechanism in the Go SDK. An Example can be found here


## Setup of the simple-fabric-client-go prototype
- create a directory for the goworkspace eg ~/code2/gow
- export GO111MODULE=on
- export GOPATH=~/code2/pow
- mkdir src
- cd src
- git clone https://github.com/davidkel/simple-fabric-client-go
- cd simple-fabric-client-go
- go mod download
- cd cmd/invoke
- go build
- ./invoke

## Issues
- There is still a bug in handling the CA entry such that TLSCaCerts is mandatory. This fixes it although the fix is rubbish and I should do a better one
file is: identityconfig.go, method is: getServerCerts

```
	for i, certPath := range certFiles {
		// THIS IS THE CA definition fix to not have to have
		// TLSCACerts defined added the if statement
		if len(certPath) > 0 {
			bytes, err := ioutil.ReadFile(pathvar.Subst(certPath))
			if err != nil {
				return nil, errors.WithMessage(err, "failed to load server certs")
			}
			serverCerts[i] = bytes
		}
	}
```
- For Alpha4 I had to change endpointconfig.go here ()
```
// EventServiceType returns the type of event service client to use
func (c *EndpointConfig) EventServiceType() fab.EventServiceType {
	etype := c.backend.GetString("client.eventService.type")
	switch etype {
	case "eventhub":
		return fab.EventHubEventServiceType
	case "deliver":
		return fab.DeliverEventServiceType
	default:
		//DAVE: return fab.AutoDetectEventServiceType commented out and replaced with
		return fab.DeliverEventServiceType
	}
}
```
I haven't had to make this change in the master branch version but this may be because I am testing against byfn rather than my own local fabric which may not have had discovery quite right

Also in chprovider.go I had this comment related to the above
```
func useDeliverEvents(ctx context.Client, chConfig fab.ChannelCfg) (bool, error) {
	switch ctx.EndpointConfig().EventServiceType() {
	case fab.DeliverEventServiceType:
		return true, nil
	case fab.EventHubEventServiceType:
		return false, nil
	case fab.AutoDetectEventServiceType:
		logger.Debug("Determining event service type from channel capabilities...")
		//DAVE: How does this work ?
		return chConfig.HasCapability(fab.ApplicationGroupKey, fab.V1_1Capability), nil
	default:
		return false, errors.Errorf("unsupported event service type: %d", ctx.EndpointConfig().EventServiceType())
	}
}
```

## BYFN Connection profile
For the master branch version here is the connection profile I used in yaml format
```yaml
---
name: first-network-org1
version: 1.0.0
client:
  organization: Org1
  connection:
    timeout:
      peer:
        endorser: '300'
organizations:
  Org1:
    cryptoPath: /tmp/byfnorg1crypto
    mspid: Org1MSP
    peers:
    - peer0.org1.example.com
    - peer1.org1.example.com
    certificateAuthorities:
    - ca.org1.example.com
peers:
  peer0.org1.example.com:
    url: grpcs://localhost:7051
    tlsCACerts:
      path: /home/dave/othercode/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/tlsca/tlsca.org1.example.com-cert.pem
    grpcOptions:
      ssl-target-name-override: peer0.org1.example.com
  peer1.org1.example.com:
    url: grpcs://localhost:8051
    tlsCACerts:
      path: /home/dave/othercode/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/tlsca/tlsca.org1.example.com-cert.pem
    grpcOptions:
      ssl-target-name-override: peer1.org1.example.com
certificateAuthorities:
  ca.org1.example.com:
    url: https://ca.org1.example.com:7054
    caName: ca-org1
    tlsCACerts:
      path: /home/dave/othercode/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/ca/ca.org1.example.com-cert.pem

channels: 
  davechannel: 
    peers: 
      peer0.org1.example.com:

    orderers:
      - orderer.example.com

orderers:
  orderer.example.com:
    url: grpcs://orderer.example.com:7050
    tlsCACerts:
      path: /home/dave/othercode/fabric-samples/first-network/crypto-config/ordererOrganizations/example.com/tlsca/tlsca.example.com-cert.pem
   
```
### No BCCSP
```json
        "BCCSP": {
            "security": {
                "enabled": true,
                "default": {
                    "provider": "SW"
                 },
                 "hashAlgorithm": "SHA2",
                 "softVerify": true,
                 "level": 256
            }
        },
```
So this section is now optional using the master branch and appears to default to this
### No Credential store
```json
        "credentialStore": {
            "path": "/tmp/state-store",
            "cryptoStore": {
                 "path": "/tmp/msp"
            }
        }
```
This means that credentials are not stored supposedly but if I don't specify this information the private key used for enrollment is actually persisted into a `keystore` directory in the working directory. It also creates an `msp` directory in the working directory but nothing is stored there.
### Requires embedded users or an entry in the orgs definition
In my above example I have had to include a cryptoPath entry the following in the org
```yaml
organizations:
  Org1:
    cryptoPath: /tmp/byfnorg1crypto
```
I haven't seen it being used as yet for my experiments but either this or embedded users are required

### Discovery
Discovery doesn't work as needed. I had to add the following channel information to get it to work
```yaml
channels: 
  davechannel: 
    peers: 
      peer0.org1.example.com:

    orderers:
      - orderer.example.com
```

Also I had to add an orderer definition because what is probably a bug in the Go SDK it doesn't get the correct tlsCA certs

```yaml
orderers:
  orderer.example.com:
    url: grpcs://orderer.example.com:7050
    tlsCACerts:
      path: /home/dave/othercode/fabric-samples/first-network/crypto-config/ordererOrganizations/example.com/tlsca/tlsca.example.com-cert.pem
```

(as a side note, if you declare the orderer in a connection profile then the node-sdk ends up with 2 orderer entries. This is a bug in the node sdk, don't know about the Go SDK)



As a final example here is my connection profile for my simple fabric in JSON format
```json
{
    "name": "basic-network",
    "version": "1.0.0",
    "client": {
        "organization": "Org1",
        "connection": {
            "timeout": {
                "peer": {
                    "endorser": "300"
                },
                "orderer": "300"
            }
        },
        "BCCSP": {
            "security": {
                "enabled": true,
                "default": {
                    "provider": "SW"
                 },
                 "hashAlgorithm": "SHA2",
                 "softVerify": true,
                 "level": 256
            }
        },
        "credentialStore": {
            "path": "/tmp/state-store",
            "cryptoStore": {
                 "path": "/tmp/msp"
            }
        }
    },
    "channels": {
        "mychannel": {
            "orderers": [
                "orderer.example.com"
            ],
            "peers": {
                "peer0.org1.example.com": {}
            }
        }
    },
    "organizations": {
        "Org1": {
            "cryptoPath": "/tmp/org1crypto",
            "mspid": "Org1MSP",
            "peers": [
                "peer0.org1.example.com"
            ],
            "certificateAuthorities": [
                "ca.org1.example.com"
            ]
        }
    },
    "orderers": {
        "orderer.example.com": {
            "url": "grpc://localhost:7050"
        }
    },
    "peers": {
        "peer0.org1.example.com": {
            "url": "grpc://localhost:7051"
        }
    },
    "certificateAuthorities": {
        "ca.org1.example.com": {
            "url": "http://localhost:7054",
            "caName": "ca.org1.example.com"
        }
    }
}
```

## Other useful observations about the Go SDK
- Go SDK uses discovery by default
- The OOB usage I tried had to send to all peers, if any were non contactable then the request failed so not sure how it worked out the peers to send to. I assume it decides to send to all rather than use endorsement plans
- No asLocalhost option so for a development fabric I had to add entries into /etc/hosts
- There appears to be more powerful capabilities such as changing the selection provider such as dynamic, round robin etc but no detail about how you might use that in the Go Doc. The CLI from secure key that uses the Go SDK is a source of example, plus if you can find the right test then there are examples in the tests.
```Go
		opts = append(opts, fabsdk.WithServicePkg(svcPackage))
	}
	opts = append(opts, fabsdk.WithCorePkg(&cryptoSuiteProviderFactory{}))


    sdk, err := fabsdk.New(cliconfig.Provider(), opts...)

	if cp.selection == nil {
		switch cliconfig.Config().SelectionProvider() {
		case cliconfig.StaticSelectionProvider:
			cliconfig.Config().Logger().Debugf("Using static selection provider.")
			cp.selection, err = staticselection.NewService(discovery)
		case cliconfig.DynamicSelectionProvider:
			cliconfig.Config().Logger().Debugf("Using dynamic selection provider.")
			cp.selection, err = dynamicselection.NewService(ctx, channelID, discovery)
		case cliconfig.FabricSelectionProvider:
			cliconfig.Config().Logger().Debugf("Using Fabric selection provider.")
			cp.selection, err = fabricselection.New(ctx, channelID, discovery)
		default:
			return nil, errors.Errorf("invalid selection provider: %s", cliconfig.Config().SelectionProvider())
        }
```
- I haven't figured out the event handling strategies yet.
- in theory query strategies are pluggable bases on the selection provider I'm guessing
- It is possible to provide your own persistence mechanism for the crypto/key store see https://github.com/hyperledger/fabric-sdk-go/blob/master/test/integration/pkg/client/msp/user_data_mgmt_test.go
- lots of things are configurable it is just a matter of working out how to build your own custom impl. Options passed are usually defined by invoking a function that starts `With`. These actually return anonymous functions which are then executed to define the option configuration (Is this a good practice pattern being employed here ?)

### CA interaction
The Go SDK doesn't support the ability to not verify the CA Server TLS Cert. You can declare verify as false but it ignores it. If you want to test CA interaction with BYFN note that BYFN uses the wrong certs for TLS. The cert to use is shown in the above yaml example to make it work.

## How to debug Go GRPC flows
general Go debug output
```bash
GODEBUG=http2debug=2
```

specific to GRPC

```bash
GRPC_GO_LOG_VERBOSITY_LEVEL=4
GRPC_GO_LOG_SEVERITY_LEVEL=INFO (that is the highest)
```

## Notes from experiments with master
```
BYFN

- failed to create SDK: failed to create identity manager provider: failed to initialize identity manager for organization: org1: Either a cryptopath or an embedded list of users is required
- need to add the following to config
organizations:
  Org1:
    cryptoPath: /tmp/byfnorg1crypto   <------
    mspid: Org1MSP

- failed to enroll the admin user: enroll failed: enroll failed: POST failure of request: POST https://localhost:7054/enroll
{"hosts":null,"certificate_request":"-----BEGIN CERTIFICATE REQUEST-----\nMIH0MIGcAgEAMBAxDjAMBgNVBAMTBWFkbWluMFkwEwYHKoZIzj0CAQYIKoZIzj0D\nAQcDQgAEvUJSZxLMjFuMkwOwBCCgAlKpt8oKhqKTfidS7SCxlt6DsFhHkZ/ojQOU\n1/1lvKmKyOt35JMMKeGNnPfgFCPOoqAqMCgGCSqGSIb3DQEJDjEbMBkwFwYDVR0R\nBBAwDoIMZGF2ZS1taW50bWFjMAoGCCqGSM49BAMCA0cAMEQCIG3b12nUyN0gsnHf\n8La6e/+D/eyi6N5jxGl7roDxcEC3AiBdL+/HlEfEZyE9otVZe3BF6Kigsc5R26jU\nJhdrB2TaAA==\n-----END CERTIFICATE REQUEST-----\n","profile":"","crl_override":"","label":"","NotBefore":"0001-01-01T00:00:00Z","NotAfter":"0001-01-01T00:00:00Z","CAName":"ca-org1"}: Post https://localhost:7054/enroll: x509: certificate is valid for ca.org1.example.com, not localhost
- more of a problem now. no verify false option supported, although it is declared.

- if credential store not provided as below, in client section looks like it uses the working directory as default
well I can find the key for the enroll persisted, but not for the other info such as cert and state store

credentialStore: 
 path: "/tmp/state-store"
 cryptoStore: 
  path: "/tmp/msp"

- no channel: event service creation failed: could not get chConfig cache reference: no channel peers configured for channel [davechannel]

added channel section
channels: 
 davechannel: 
  peers: 
    peer0.org1.example.com:


Does use service discovery by default (not sure how you override this as yet)

debug go grpc calls
GODEBUG=http2debug=2

GRPC_GO_LOG_VERBOSITY_LEVEL=4
GRPC_GO_LOG_SEVERITY_LEVEL=INFO (that is the highest)


With Go I ensured I couldn't work with a peer from each org and it keeps failing to connect as it chooses them and doesn't 
handle it.

Problems as well with discovery and the tlsCA cert for the orderer. It works if I define the orderer in the connection profile otherwise it doesn't
```


## Old notes from experiments with alpha4
- set the GOPATH
- go get -u github.com/golang/dep/cmd/dep
- ../bin/dep ensure
- cd fabric-sdk-go/
-  ../../../../bin/dep ensure
vi Gopkg.toml
  180  ls
  181  rm -fr github.com/
  182  rm -fr ../pkg/
  183  ../bin/dep ensure
  184  mkdir code
  185  mv * code
  186  ls
  187  cd code
  188  ../../bin/dep ensure


export GOPATH=~/my-github-repos/sandbox/gosdk
export PATH=/usr/local/go/bin:$PATH
../../bin/dep ensure
fix test.go
go build test.go
find fix for sdk code: identityconfig.go, getServerCerts

```go
	for i, certPath := range certFiles {
		// THIS IS THE CA definition fix to not have to have
		// TLSCACerts defined added the if statement
		if len(certPath) > 0 {
			bytes, err := ioutil.ReadFile(pathvar.Subst(certPath))
			if err != nil {
				return nil, errors.WithMessage(err, "failed to load server certs")
			}
			serverCerts[i] = bytes
		}
	}
```
