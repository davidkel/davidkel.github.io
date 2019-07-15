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




