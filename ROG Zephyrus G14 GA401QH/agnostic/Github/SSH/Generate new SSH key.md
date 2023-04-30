## About SSH key passphrases

You can access and write data in repositories on GitHub.com using SSH (Secure Shell Protocol). When you connect via SSH, you authenticate using a private key file on your local machine. For more information, see "[About SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/about-ssh)."

When you generate an SSH key, you can add a passphrase to further secure the key. Whenever you use the key, you must enter the passphrase. If your key has a passphrase and you don't want to enter the passphrase every time you use the key, you can add your key to the SSH agent. The SSH agent manages your SSH keys and remembers your passphrase.

If you don't already have an SSH key, you must generate a new SSH key to use for authentication. If you're unsure whether you already have an SSH key, you can check for existing keys. For more information, see "[Checking for existing SSH keys](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/checking-for-existing-ssh-keys)."

If you want to use a hardware security key to authenticate to GitHub, you must generate a new SSH key for your hardware security key. You must connect your hardware security key to your computer when you authenticate with the key pair. For more information, see the [OpenSSH 8.2 release notes](https://www.openssh.com/txt/release-8.2).

## [](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key)Generating a new SSH key

You can generate a new SSH key on your local machine. After you generate the key, you can add the key to your account on GitHub.com to enable authentication for Git operations over SSH.

**Note:** GitHub improved security by dropping older, insecure key types on March 15, 2022.

As of that date, DSA keys (`ssh-dss`) are no longer supported. You cannot add new DSA keys to your personal account on GitHub.com.

RSA keys (`ssh-rsa`) with a `valid_after` before November 2, 2021 may continue to use any signature algorithm. RSA keys generated after that date must use a SHA-2 signature algorithm. Some older clients may need to be upgraded in order to use SHA-2 signatures.

1.  Open Terminal.
    
2.  Paste the text below, substituting in your GitHub email address.
    
    ```shell
    $ ssh-keygen -t ed25519 -C "your_email@example.com"
    ```
    
    **Note:** If using *pass* as your password manager, you can use `ssh-keygen -t ed25519 -C "[EMAIL] -P $(pass [PASSWORD FILE])"` instead
     
    **Note:** If you are using a legacy system that doesn't support the Ed25519 algorithm, use:
    
    ```shell
    $ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
    ```
    
    This creates a new SSH key, using the provided email as a label.
    
    ```shell
    > Generating public/private ALGORITHM key pair.
    ```
    
    When you're prompted to "Enter a file in which to save the key", you can press **Enter** to accept the default file location. Please note that if you created SSH keys previously, ssh-keygen may ask you to rewrite another key, in which case we recommend creating a custom-named SSH key. To do so, type the default file location and replace id_ssh_keyname with your custom key name.
    
    ```shell
    > Enter a file in which to save the key (/home/YOU/.ssh/ALGORITHM):[Press enter]
    ```
    
3.  At the prompt, type a secure passphrase. For more information, see "[Working with SSH key passphrases](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/working-with-ssh-key-passphrases)."
    
    ```shell
    > Enter passphrase (empty for no passphrase): [Type a passphrase]
    > Enter same passphrase again: [Type passphrase again]
    ```
    

## [](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#adding-your-ssh-key-to-the-ssh-agent)Adding your SSH key to the ssh-agent

Before adding a new SSH key to the ssh-agent to manage your keys, you should have checked for existing SSH keys and generated a new SSH key.

1.  Start the ssh-agent in the background.
    
    ```shell
    $ eval "$(ssh-agent -s)"
    > Agent pid 59566
    ```
    
    Depending on your environment, you may need to use a different command. For example, you may need to use root access by running `sudo -s -H` before starting the ssh-agent, or you may need to use `exec ssh-agent bash` or `exec ssh-agent zsh` to run the ssh-agent.
    
2.  Add your SSH private key to the ssh-agent. If you created your key with a different name, or if you are adding an existing key that has a different name, replace _id_ed25519_ in the command with the name of your private key file.
    
    ```shell
    $ ssh-add ~/.ssh/id_ed25519
    ```
    
3.  Add the SSH key to your account on GitHub. For more information, see "[Adding a new SSH key to your GitHub account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)."
    
    **Tip:** Use `ssh-add -L` to confirm added identities.

## [](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key-for-a-hardware-security-key)Generating a new SSH key for a hardware security key

If you are using macOS or Linux, you may need to update your SSH client or install a new SSH client prior to generating a new SSH key. For more information, see "[Error: Unknown key type](https://docs.github.com/en/authentication/troubleshooting-ssh/error-unknown-key-type)."

1.  Insert your hardware security key into your computer.
    
2.  Open Terminal.
    
3.  Paste the text below, substituting in the email address for your account on GitHub.
    
    ```shell
    $ ssh-keygen -t ed25519-sk -C "YOUR_EMAIL"
    ```
    
    **Note:** If the command fails and you receive the error `invalid format` or `feature not supported,` you may be using a hardware security key that does not support the Ed25519 algorithm. Enter the following command instead.
    
    ```shell
    $ ssh-keygen -t ecdsa-sk -C "your_email@example.com"
    ```
    
4.  When you are prompted, touch the button on your hardware security key.
    
5.  When you are prompted to "Enter a file in which to save the key," press Enter to accept the default file location.
    
    ```shell
    > Enter a file in which to save the key (/home/YOU/.ssh/id_ed25519_sk):[Press enter]
    ```
    
6.  When you are prompted to type a passphrase, press **Enter**.
    
    ```shell
    > Enter passphrase (empty for no passphrase): [Type a passphrase]
    > Enter same passphrase again: [Type passphrase again]
    ```
    
7.  Add the SSH key to your account on GitHub. For more information, see "[Adding a new SSH key to your GitHub account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)."