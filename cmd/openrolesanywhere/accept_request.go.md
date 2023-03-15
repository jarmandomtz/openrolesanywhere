# accept_requet.go

Pre requisites

- 

Actions

- Format the Certificate serial number to string
- Get context (aws config configuration)
- Creates a KMS api using context
- Reads the ca.pem CA certificate
- Converts the certificate to x509 Certificate type for use the CA URIs
- Finds the KMS ARN on the Certificate URIs
- Gets the IAM Roles Anywhere Profile from "profile-arn.txt
- Gets the IAM Roles Anywhere Trust Anchor from "trust-anchor-arn.txt
- Generate a KMSSIGNER using context, KMS API and Key ARN
- Read the User Certificate/Public Key and get the Public Key in PEM format}
- Get a SubjectPublicKeyInfo with the Public Key (function x509.ParsePKIXPublicKey)
- Creates a X509 structure adding serial number, subject, period, key usage, and URIs from profile and trust anchor
- Creates a certificate bytes passing: X590 structure (previous step), CA (ca.pem), public key (user key) and signer (using KMS key)
- Encode bytes to PEM format on Memory CERTIFICATE type and print it to the console