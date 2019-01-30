# Secure SSH setup

## Use key-based login

Copy your public key from the client machine to the server. Run the following command on the client:

```console
ssh-copy-id -i ~/.ssh/my_key user@server
```
