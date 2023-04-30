## About addition of SSH keys to your account

You can access and write data in repositories on GitHub.com using SSH (Secure Shell Protocol). When you connect via SSH, you authenticate using a private key file on your local machine. For more information, see "[About SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/about-ssh)."

You can also use SSH to sign commits and tags. For more information about commit signing, see "[About commit signature verification](https://docs.github.com/en/authentication/managing-commit-signature-verification/about-commit-signature-verification)."

After you generate an SSH key pair, you must add the public key to GitHub.com to enable SSH access for your account.

## [](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account#prerequisites)Prerequisites

Before adding a new SSH key to your account on GitHub.com, complete the following steps.

1.  Check for existing SSH keys. For more information, see "[Checking for existing SSH keys](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/checking-for-existing-ssh-keys)."
2.  Generate a new SSH key and add it to your machine's SSH agent. For more information, see "[Generating a new SSH key and adding it to the ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)."

## [](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account#adding-a-new-ssh-key-to-your-account)Adding a new SSH key to your account

After adding a new SSH authentication key to your account on GitHub.com, you can reconfigure any local repositories to use SSH. For more information, see "[Managing remote repositories](https://docs.github.com/en/get-started/getting-started-with-git/managing-remote-repositories#switching-remote-urls-from-https-to-ssh)."

**Note:** GitHub improved security by dropping older, insecure key types on March 15, 2022.

As of that date, DSA keys (`ssh-dss`) are no longer supported. You cannot add new DSA keys to your personal account on GitHub.com.

RSA keys (`ssh-rsa`) with a `valid_after` before November 2, 2021 may continue to use any signature algorithm. RSA keys generated after that date must use a SHA-2 signature algorithm. Some older clients may need to be upgraded in order to use SHA-2 signatures.

1.  Copy the SSH public key to your clipboard.
    
    If your SSH public key file has a different name than the example code, modify the filename to match your current setup. When copying your key, don't add any newlines or whitespace.
    
    ```shell
    $ cat ~/.ssh/id_ed25519.pub
      # Then select and copy the contents of the id_ed25519.pub file
      # displayed in the terminal to your clipboard
    ```
    
    **Tip:** If using an *X11* server, you can use `xclip -selection clipboard id_25519.pub` to copy the contents to the clipboard.
    
    **Tip:** Alternatively, you can locate the hidden `.ssh` folder, open the file in your favorite text editor, and copy it to your clipboard.
    
2.  In the upper-right corner of any page, click your profile photo, then click **Settings**.
    
    ![Screenshot of GitHub's account menu showing options for users to view and edit their profile, content, and settings. The menu item "Settings" is outlined in dark orange.](https://docs.github.com/assets/cb-139735/images/help/settings/userbar-account-settings.png)
    
3.  In the "Access" section of the sidebar, click  **SSH and GPG keys**.
    
4.  Click **New SSH key** or **Add SSH key**.
    
5.  In the "Title" field, add a descriptive label for the new key. For example, if you're using a personal laptop, you might call this key "Personal laptop".
    
6.  Select the type of key, either authentication or signing. For more information about commit signing, see "[About commit signature verification](https://docs.github.com/en/authentication/managing-commit-signature-verification/about-commit-signature-verification)."
    
7.  In the "Key" field, paste your public key.
    
8.  Click **Add SSH key**.
    
9.  If prompted, confirm access to your account on GitHub. For more information, see "[Sudo mode](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/sudo-mode)."