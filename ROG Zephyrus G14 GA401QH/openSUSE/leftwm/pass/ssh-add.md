# ISSUE:
- ## openSUSE's version of \`ssh-add\` does not include a passphrase option to use with \`pass\`


# SOLUTION:
- ## Hack the \`SSH_ASKPASS\` and \`SSH_ASKPASS_REQUIRE\` environment variables to have \`ssh-add\` use a script as the password helper

*~/.zshrc*
```bash
export SSH_ASKPASS_REQUIRE="force"

SSH_ASKPASS="/path/to/script/pass.sh" ssh-add ~/.ssh/id_rsa < /dev/null
```

*pass.sh*
```bash
#!/bin/sh
echo $(pass [PASSWORD FILE])
```