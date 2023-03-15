# create-ca

Pre requisites of steps done previously

- getX509Name(): Gets certificate info from parameters: serial number, organization, organization unit, country, locality, province, street, postal code


Actions,
- Receives: context, keyId, subject (all cert params), duration
- Get certificate serial number 
- Get context
- Creates KMS api
- Get KMS data throw KMS api using context, keyId
- Get KMS arn using KMS data
- Creates KMSSIGNER api using KMS api and Key ARN
- Creates a x509 certificate variable called CA
- Creates a certificate using CA and KMSSIGNER in raw format
- Creates a certificate in PEM format, name it ca.pem and permissions 0700
- Creates a ROLESANYWHERE api
- Creates a TRUSTANCHOR using ROLESANYWHERE api, passing params: name, sourceData, sourceType, Tags, certPEM
- Generate "trust-anchor-arn.txt" file with arn, permissions 0700