# request_certificate.go
Use an SSH key fingerprint to request an X.509 certificate from an admin local user

Pre conditions,

- Was obtained the public SSH key fingerprint from the local user

Actions,

- Retrieves the User Public key Fingerprint from the command line arguments: ssh-fingerprint
- Calls signerFromFingerprint passing the fingerprint

main.go -> signerFromFingerprint >>
- Get SSH_AUTH_SOCK variable from local environment, in my case "/run/user/1000/keyring/ssh", more info [here](https://unix.stackexchange.com/questions/424520/how-is-my-ssh-key-unlocked-without-ssh-agent-and-how-do-i-fix-that) and [wiki for gnome-keyring-daemon](https://wiki.gnome.org/Projects/GnomeKeyring/Ssh)
- Open a socket/connection to the ssh service (which keeps ssh keys opened in memory) and generates a client
- With the client generates a list of signers, and iterate converting the Public key to SHA256/fingerprint and compare with the one fingerprint passed as parameter
- When it find it, get the AlgorithmSigner and execute openrolesanywhere.NewSignerFromSshSigner

ssh_signer.go -> NewSignerFromSshSigner
- Builds an sshSinger structure using the previous AlgorithmSigner found
<< main.go -> signerFromFingerprint

- x509.MarshalPKIXPublicKey converts a public key to PKIX, ASN.1 DER form. The encoded public key is a SubjectPublicKeyInfo structure . PKIX = Public Key Infrastructure for X.509 Certificates (IETF: Internet Engineering Task Force)
- Creates a PEM encoding Certificate from the Public key on DER format to Memory and print it to the console

## Helpers

How to export Public key in PEM format from a keypair file id_rsa

```shell
%> ssh-keygen -f ~/.ssh/id_rsa -e -m pem
-----BEGIN RSA PUBLIC KEY-----
MIIBigKCAYEAl9cTGeFv8GMCPFv5yNYeL5jl4oPQw/NL2ryPW8DtSdNpaieh2WB/
1gqiI93bskBJKZpaHQUMg9iXGYi+VAulHX+EzkUTG/KPJdECBaKEgFMN9DbkxvUv
/mlOj1YLejJ4TA6t9byTrADDX3UAGSpRbxL8OdPGG2S4H0PujK5KByjkWNiWdAHb
uotNzdZlqacPiwFP33xYHoJe2lS36D0Op2m/bbd+Oe4jiZDgMohKu/j+NjTRNFM6
hFqE121lHaLAI67ouREnAYWx9rR1YB6dPS3QuiVr5VEUukqCWLGjM/KyJPo/170p
79F2i7zapLyJvLOc1w5iANMXpJjTGOF71hMlXyIV61nyiYQp8aWXHZaxZeMrNRXP
I5GQDaK+m+Xvz2HwEF2/7/WXR3eyjQbVX60nrQNJNoMRrBkVXolZGnT62s3gam2O
w1xXYAUg7HkE/9OtUCqqVAQ2I3khYVecnNdJRb16+gLSU/ThX6EO4Z5QiO/Jvg+y
tXDm+RVKdpInAgMBAAE=
-----END RSA PUBLIC KEY-----
```

How to export Public key in PEM format from a public key file id_rsa.pub RSA format

```shell
%> ssh-keygen -f ~/.ssh/id_rsa.pub -e -m pem
-----BEGIN RSA PUBLIC KEY-----
MIIBigKCAYEAl9cTGeFv8GMCPFv5yNYeL5jl4oPQw/NL2ryPW8DtSdNpaieh2WB/
1gqiI93bskBJKZpaHQUMg9iXGYi+VAulHX+EzkUTG/KPJdECBaKEgFMN9DbkxvUv
/mlOj1YLejJ4TA6t9byTrADDX3UAGSpRbxL8OdPGG2S4H0PujK5KByjkWNiWdAHb
uotNzdZlqacPiwFP33xYHoJe2lS36D0Op2m/bbd+Oe4jiZDgMohKu/j+NjTRNFM6
hFqE121lHaLAI67ouREnAYWx9rR1YB6dPS3QuiVr5VEUukqCWLGjM/KyJPo/170p
79F2i7zapLyJvLOc1w5iANMXpJjTGOF71hMlXyIV61nyiYQp8aWXHZaxZeMrNRXP
I5GQDaK+m+Xvz2HwEF2/7/WXR3eyjQbVX60nrQNJNoMRrBkVXolZGnT62s3gam2O
w1xXYAUg7HkE/9OtUCqqVAQ2I3khYVecnNdJRb16+gLSU/ThX6EO4Z5QiO/Jvg+y
tXDm+RVKdpInAgMBAAE=
-----END RSA PUBLIC KEY-----

%> 
```

How to enable/configure authentication on ssh/sftp using ssh keypair

```shell
#Generate ssh keypair
%> ssh-keygen -t rsa -b 4086 -C usersftp@localhost -f userftp -N srmTG304y
%> ls -la ~/.ssh/u*
-rw------- 1 operatoruser operatoruser 3430 Sep 24 17:18 userftp
-rw-r--r-- 1 operatoruser operatoruser  740 Sep 24 17:18 userftp.pub
#Pass the public key to the service administrator and ask to add on known_hosts file
%> cat /home/armando/.ssh/known_hosts
|1|jwA5NRrsn+De4YOOHzOuNY8gEig=|bXNIinzI2HQSNGMaPctwR/O8rqk= ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBAwXIqM2bXzQlfU+N2ZrBWTCIRkDw8RkqwiFjCjsOev4J9VVvc8qCjF4X2BnsMewiSLaUyFg1xEnaKzQ3NcsN3s=
|1|zoRSjQXULhWoZtspT5tT/InSSw4=|8UwRK1vIRKqRwFnTnqk1HDidAaE= ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAq2A7hRGmdnm9tUDbO9IDSwBK6TbQa+PXYPCPy6rbTrTtw7PHkccKrpp0yVhp5HdEIcKr6pLlVDBfOLX9QUsyCOV0wzfjIJNlGEYsdlLJizHhbn2mUjvSAHQqZETYP81eFzLQNnPHt4EVVUh7VfDESU84KezmD5QlWpXLmvU31/yMf+Se8xhHTvKSCZIFImWwoG6mbUoWf9nzpIoaSjB+weqqUUmpaaasXVal72J+UX2B+2RPW3RcT0eOzQgqlJL3RKrTJvdsjE3JEAvGq3lGHSZXy28G3skua2SmVi/w4yCE6gbODqnTWlg7+wC604ydGXA8VJiS5ap43JXiUFFAaQ==
|1|cURXlk2JOMEyhh479aTrvmDbQUo=|sr0JYEBgvuFguHcrPm44p8w1XuI= ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDGAhjW0JFAibHZWCoQI7FkD+hHn1oliwkKPZANDMzVa
```
