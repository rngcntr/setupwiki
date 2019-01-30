# TODO

```console
ssh-copy-id -i ~/.ssh/«key» «user»@«hostname»
```

```
/etc/ssh/sshd_config

PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM no
```

```console
# service ssh restart
```
