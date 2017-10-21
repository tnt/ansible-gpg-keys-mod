# ansible-gpg-import

Ansible module to import and remove GPG-keys.

It addresses the issues of non-responding keyservers in general by repeating attempts and keys.gnupg.net's round-robin-DNS sabotaging DNS-caching (like the Windows DNS-cache does it) in particular by optionally trying alternating hostnames.

### examples

```YAML
- name: Install GPG key
  gpg_import: key_id="0x3804BB82D39DC0E3" state=present

- name: Install GPG key
  gpg_import:
    key_id: "0x3804BB82D39DC0E3"
    bin_path: '/usr/local/bin/gpg'

- name: Install or update GPG key
  gpg_import:
    key_id: "0x3804BB82D39DC0E3"
    state: latest
    servers:
      - 'hkp://no.way.ever'
      - 'keys.gnupg.net'
      - 'hkps://hkps.pool.sks-keyservers.net'

- name: Install or fail with fake and not fake GPG keys
  gpg_import:
    key_id: "{{ item }}"
    tries: 2
  with_items:
    - "0x3804BB82D39DC0E3"
    - "0x3804BB82D39DC0E4" # fake key fails

- name: import a file-based public key
  gpg_import: key_type=public state=present key_file=/etc/customer-key/customer.pubkey

- name: import a file-based private key
  gpg_import: key_type=private state=present key_file=/etc/customer-key/customer.privatekey
```

### options
name         | default            | description
-------------|:------------------:|-------------
key_id       | null               | The id of the key to be fetched and imported. Only applicable to public keys. Either key_file or key_id is required.
key_file     | null               | Filename of key to be imported. Must be on remote machine, not local. Either key_file or key_id is required.
key_type     | 'private'          | What type of key to import. Only applicable to key_file
bin_path     | /usr/bin/gpg       | Location of gpg binary
state        | 'present'          | Desired state 'present', 'latest', 'refreshed' or 'absent' ('refreshed' == 'latest')
servers      | [ keys.gnupg.net ] | List of hostnames (or `hkp://`/`hkps://` urls) to try
tries        | 3                  | Number of attempts per *server*
delay        | 0.5                | Delay between retries
gpg_timeout  | 5                  | Timeout parameter for gpg

[Strange behaviors](https://gist.github.com/tnt/eedaed9a6cc75130b9cb) occur when used with [insane keys](https://gist.github.com/tnt/70b116c72be11dc3cc66). But this is a gpg-problem.
