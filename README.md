# vax

Vax is a Golang AWS credentials provider using the Hashicorp Vault AWS secret
engine.

Vax pulls STS credentials from the configured path, and passes them back to
an AWS SDK credentials object by implementing the `credentials.Provider`
interface.

## Usage

This example assumes you wish to pull credentials from a role named `myrole`
in an AWS secrets engine mounted at `aws`, and that you have set the env vars
`VAULT_ADDR` and `VAULT_TOKEN` with the appropriate values:

    package main

    import (
    	"github.com/aws/aws-sdk-go/aws"
    	"github.com/aws/aws-sdk-go/aws/credentials"
    	"github.com/aws/aws-sdk-go/aws/session"
    	"github.com/aws/aws-sdk-go/service/sts"
    	"github.com/daveadams/vax"
    	"log"
    )

    const (
        SecretsEngineMount = "aws"
        EngineRoleName     = "myrole"
    )

    func main() {
        stsSvc := sts.New(session.Must(session.NewSession()), &aws.Config{
    	    Credentials: credentials.NewCredentials(
                vax.NewVaultProvider(SecretsEngineMount, EngineRoleName),
            ),
        })

        resp, err := stsSvc.GetCallerIdentity(&sts.GetCallerIdentityInput{})
        if err != nil {
            log.Fatalf("ERROR: %s", err)
        }

        log.Printf("Hello, %s from account %s\n", *resp.Arn, *resp.Account)
    }

The provider will seamlessly request new credentials from the provider whenever
they expire. So long as the Vault session tied to the Vault token itself does
not expire, the credentials should continue to be valid.

## License

This software is public domain. No rights are reserved. See LICENSE for more
information.
