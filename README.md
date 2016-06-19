# Ansible

Automated server setup for [Plotline](https://github.com/pch/plotline) apps.

## Contents

It sets up:

* system user accounts: admin (sudoer), deploy (regular)
* automatic security updates (unattended upgrades)
* iptables-based firewall & fail2ban
* nginx
* ruby 2.3.0 (using rbenv)
* rails app directories, nginx vhost and database config
* postgres 9.4
* thumbor as an image thumbnailing service
* runit for process supervision
* Amazon S3 backups (for the database and uploaded files)
* dropbox (headless)
* logrotate
* swap (1GB by default)

## Usage

### Site configuration

```
cp group_vars/all/custom-example group_vars/all/custom
```

Now edit the `group_vars/all/custom` file and change the values to match your
needs.

### Passwords and Sensitive Information

Passwords and other sensitive information should be stored in an encrypted
vault file:

```
ansible-vault create group_vars/all/vault
ansible-vault edit group_vars/all/vault
```

You should now be able to edit the vault file. Paste the example below
to get started:

```
vault_common_public_key: "{{ lookup('file', '/Users/MY_USER/.ssh/id_rsa.pub') }}"

vault_admin_password: ADMIN_PASS
vault_deploy_password: DEPLOY_PASS

vault_postgres_password: DATABASE_PASSWORD

vault_app_secret_key_base: RANDOM_SECRET # what will be put in config/secrets.yml

vault_thumbor_security_key: 'RANDOMSTRING'

vault_aws_access_key: 'AWS_ACCESS_KEY'
vault_aws_secret_key: 'AWS_SECRET'
```

To generate system users' passwords, run:

```
python -c "from passlib.hash import sha512_crypt; import getpass; print sha512_crypt.encrypt('YOURPASS')"
```

Paste the output to the vault file, replacing the value of `vault_admin_password`
and `vault_deploy_password` (each user should have a different password!).

## Dependencies

Run the following command to install the logrotate dependency:

```bash
ansible-galaxy install nickhammond.logrotate
```

## Run Playbook

Put the IP of your server in the `hosts` file and run:

```bash
ansible-playbook playbook.yml -i hosts -u root --ask-vault-pass
```

This will take a while. Once it's complete, root access will be disabled, so
if you need to run the playbook again, use the `admin` user instead:

```bash
ansible-playbook playbook.yml -i hosts -u admin --ask-vault-pass -K
```

## Dropbox Setup

Unfortunately, in order to link the server to your Dropbox account, there's
a manual step involved:

```bash
ansible -i hosts --ask-vault-pass -m shell -a 'tail /home/deploy/dropbox-log/current' -u deploy app
```

You should see something like:

```
2016-06-18_12:09:02.37040 This computer isn't linked to any Dropbox account...
2016-06-18_12:09:02.48219 Please visit https://www.dropbox.com/cli_link_nonce?nonce=TOKEN to link this device.
```

Copy the URL, paste it to your browser and your Dropbox account should be linked.
To verify this, run the same command again and you should see:

```
2016-06-18_12:10:33.93356 This computer is now linked to Dropbox. Welcome
```

## Credits

This playbook is heavily based on [Ansible Infractructure](https://github.com/Servers-for-Hackers/ansible-infrastructure)
by [@fideloper](https://serversforhackers.com).
