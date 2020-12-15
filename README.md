# ansible-role-x509_certificate

Manages X509 private and/or public keys and CA certificates in the Linux system trust store.
The role assumes you already have valid private key or *signed* public key. The role does not create or manage CSR.

This role is based on [ansible-role-x509_certificate](https://github.com/trombik/ansible-role-x509_certificate)

And has been extended with ideas from the following roles:
 - [ansible.ca-certificates](https://github.com/arillso/ansible.ca-certificates)
 - [ansible-role-ssl-certificate](https://github.com/ome/ansible-role-ssl-certificate)

Thanks to all!

# Requirements

[debops-ansible-plugins](https://docs.debops.org/en/master/ansible/roles/ansible_plugins/index.html#debops-ansible-plugins)

# Role Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `x509_certificate__public_dir` | path to default directory to keep public certificates | `{{ __x509_certificate__public_dir }}` |
| `x509_certificate__ca_dir` | path to default directory to keep ca certificates | `{{ __x509_certificate__ca_dir }}` |
| `x509_certificate__private_dir` | path to default directory to keep private keys | `{{ __x509_certificate__private_dir }}` |
| `x509_certificate__packages` | list of packages to install to manage keys, i.e. validating certificates | `{{ __x509_certificate__packages }}` |
| `x509_certificate__public_default_owner` | default public owner of keys | `{{ __x509_certificate__public_default_owner }}` |
| `x509_certificate__public_default_group` | default public group of keys | `{{ __x509_certificate__public_default_group }}` |
| `x509_certificate__public_default_mode` | default public mode of keys | `{{ __x509_certificate__public_default_mode }}` |
| `x509_certificate__ca_default_owner` | default ca owner of keys | `{{ __x509_certificate__ca_default_owner }}` |
| `x509_certificate__ca_default_group` | default ca group of keys | `{{ __x509_certificate__ca_default_group }}` |
| `x509_certificate__ca_default_mode` | default ca mode of keys | `{{ __x509_certificate__ca_default_mode }}` |
| `x509_certificate__private_default_owner` | default private owner of keys | `{{ __x509_certificate__private_default_owner }}` |
| `x509_certificate__private_default_group` | default private group of keys | `{{ __x509_certificate__private_default_group }}` |
| `x509_certificate__private_default_mode` | default private mode of keys | `{{ __x509_certificate__private_default_mode }}` |
| `x509_certificate__additional_packages` | list of additional packages to install. they are installed before managing certificates and keys. useful when the owner of the files does not exist, but to be created by other role or task later. using this variable needs care. when a package is installed by this role, package installation task after this role will not be triggered, which might cause unexpected effects. in that case, create the user and the group by yourself | `[]` |
| `x509_certificate__validate_command` | command to validate certificate, keys and ca. The command must be defined in `x509_certificate__validate_command_private`, `x509_certificate__validate_command_ca` and `x509_certificate__validate_command_public` as key | `openssl` |
| `x509_certificate__validate_command_private` | dict of command to validate private key (see below) | `{"openssl"=>"openssl rsa -check -in %s"}` |
| `x509_certificate__validate_command_ca` | dict of command to validate ca key (see below) | `{{ x509_certificate__validate_command_public }}` |
| `x509_certificate__validate_command_public` | dict of command to validate public key (see below) | `{"openssl"=>"openssl x509 -noout -in %s"}` |
| `x509_certificate__ca_handler_create` | command to update system trust store when a new ca certificate was created | `{{ __x509_certificate__ca_handler_create }}` |
| `x509_certificate__ca_handler_remove` | command to update system trust store when a new ca certificate was removed | `{{ __x509_certificate__ca_handler_remove }}` |
| `x509_certificate__debug_log` | enable logging of sensitive data during the play if `yes`. note that the log will display the value of `x509_certificate`, including private key, if `yes` | `no` |
| `x509_certificate__dependent` | keys to manage (see below) (dependent) | `[]` |
| `x509_certificate` | keys to manage (see below) | `[]` |
| `x509_certificate__group` | keys to manage (see below) (group) | `[]` |
| `x509_certificate__host` | keys to manage (see below) (host) | `[]` |
| `x509_certificate__combined` | keys to manage (see below) (combined) | `{{ x509_certificate__dependent + x509_certificate + x509_certificate__group + x509_certificate__host }}` |

## `x509_certificate__validate_command_private` & `x509_certificate__validate_command_ca` & `x509_certificate__validate_command_public`

This variable is a dict. The key is command name and the value is used to
validate private / ca / public key files when creating.

## `x509_certificate__ca_handler_create` & `x509_certificate__ca_handler_remove`

This variable is a string. This command will be run when a new ca certificate was created / removed to the system trust store.

## `x509_certificate` & `x509_certificate__dependent` & `x509_certificate__group` & `x509_certificate__host`

All of the variables will be merged together as in the order given in `x509_certificate__combined`. This follow the DebOps best practice.
[DebOps dependent-variables](https://docs.debops.org/en/master/dep/dep-0002.html#dependent-variables)

This variable is a list of dict. Keys and Values are explained below.

| Key | Value | Mandatory? |
|-----|-------|------------|
| `name` | Descriptive name of keys | yes |
| `state` | one of `present` or `absent`. the role creates the key when `present` and removes the key when `absent` | yes |
| `public` | a dict that represents a public certificate | no |
| `ca` | a dict that represents a ca certificate | no |
| `private` | a dict that represents a private key | no |

### `public`, `private` and `ca` in `x509_certificate`

`public`, `private` and `ca` must contain a dict. The dict is explained below.

| Key | Value | Mandatory? |
|-----|-------|------------|
| `path` | path to the file. if not defined, the file will be created under `x509_certificate__dir` | no |
| `owner` | owner of the file (default is `x509_certificate__default_owner`) | no |
| `group` | group of the file (default is `x509_certificate__default_group`) | no |
| `mode` | permission of the file (default is `x509_certificate__default_mode`) | no |
| `key` | the content of the key | no |

## OS specific variables

Have a look at the `yml` files in the `vars` dir.
! FreeBSD and OpenBSD are not fully implemented yet !

# Listeners/Handlers

This role notifies a listener `x509_certificate certificate changed` when any changes on public or private keys are made.
It will NOT be notified when a change on an ca certificate are made!
This can be used to trigger a restart of any services dependent on the certificates.

For example handlers see below.

If a change are made, the variable `_x509_changed_certificates` will be set.
This variable contain a list of changed public and/or private keys with the name, path and state.
You can use this variable for fancy `when` constructs.

## Example `_x509_changed_certificates` content

```yaml
_x509_changed_certificates:
  - name: foo
    path: /etc/ssl/public/foo.crt
    state: present
  - name: bar
    path: /usr/local/etc/ssl/bar/bar.key
    state: present
```

# Dependencies

None

# Example Playbook

```yaml
- hosts: localhost
  roles:
    - ansible-role-x509_certificate
    - ansible-role-postfix
  vars:
    # XXX NEVER set this variable to `yes` unless you know what you are doing.
    x509_certificate__debug_log: yes

    x509_certificate__additional_packages:
      - quagga
    x509_certificate:
      - name: foo
        state: present
        public:
          key: |
            -----BEGIN CERTIFICATE-----
            MIIDOjCCAiICCQDaGChPypIR9jANBgkqhkiG9w0BAQUFADBfMQswCQYDVQQGEwJB
            VTETMBEGA1UECAwKU29tZS1TdGF0ZTEhMB8GA1UECgwYSW50ZXJuZXQgV2lkZ2l0
            cyBQdHkgTHRkMRgwFgYDVQQDDA9mb28uZXhhbXBsZS5vcmcwHhcNMTcwNzE4MDUx
            OTAxWhcNMTcwODE3MDUxOTAxWjBfMQswCQYDVQQGEwJBVTETMBEGA1UECAwKU29t
            ZS1TdGF0ZTEhMB8GA1UECgwYSW50ZXJuZXQgV2lkZ2l0cyBQdHkgTHRkMRgwFgYD
            VQQDDA9mb28uZXhhbXBsZS5vcmcwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEK
            AoIBAQDZ9nd1isoGGeH4OFbQ6mpzlldo428LqEYSH4G7fhzLMKdYsIqkMRVl1J3s
            lXtsMQUUP3dcpnwFwKGzUvuImLHx8McycJKwOp96+5XD4QAoTKtbl59ZRFb3zIjk
            Owd94Wp1lWvptz+vFTZ1Hr+pEYZUFBkrvGtV9BoGRn87OrX/3JI9eThEpksr6bFz
            QvcGPrGXWShDJV/hTkWxwRicMMVZVSG6niPusYz2wucSsitPXIrqXPEBKL1J8Ipl
            8dirQLsH02ZZKcxGctEjlVgnpt6EI+VL6fs5P6A45oJqWmfym+uKztXBXCx+aP7b
            YUHwn+HV4qzZQld80PSTk6SS3hMXAgMBAAEwDQYJKoZIhvcNAQEFBQADggEBAKgf
            x3K9GHDK99vsWN8Ej10kwhMlBWBGuM0wkhY0fbxJ0gW3sflK8z42xMc2dhizoYsY
            sLfN0aylpN/omocl+XcYugLHnW2q8QdsavWYKXqUN0neIMr/V6d1zXqxbn/VKdGr
            CD4rJwewBattCIL4+S2z+PKr9oCrxjN4i3nujPhKv/yijhrtV+USw1VwuFqsYaqx
            iScC13F0nGIJiUVs9bbBwBKn1c6GWUHHiFCZY9VJ15SzilWAY/TULsRsHR53L+FY
            mGfQZBL1nwloDMJcgBFKKbG01tdmrpTTP3dTNL4u25+Ns4nrnorc9+Y/wtPYZ9fs
            7IVZsbStnhJrawX31DQ=
            -----END CERTIFICATE-----
      - name: bar
        state: present
        public:
          path: /usr/local/etc/ssl/bar/bar.pub
          owner: "{% if ansible_os_family == 'FreeBSD' or ansible_os_family == 'OpenBSD' %}www{% elif ansible_os_family == 'RedHat' %}ftp{% else %}www-data{% endif %}"
          group: "{% if ansible_os_family == 'FreeBSD' or ansible_os_family == 'OpenBSD' %}www{% elif ansible_os_family == 'RedHat' %}ftp{% else %}www-data{% endif %}"
          mode: "0644"
          key: |
            -----BEGIN CERTIFICATE-----
            MIIDOjCCAiICCQDaGChPypIR9jANBgkqhkiG9w0BAQUFADBfMQswCQYDVQQGEwJB
            VTETMBEGA1UECAwKU29tZS1TdGF0ZTEhMB8GA1UECgwYSW50ZXJuZXQgV2lkZ2l0
            cyBQdHkgTHRkMRgwFgYDVQQDDA9mb28uZXhhbXBsZS5vcmcwHhcNMTcwNzE4MDUx
            OTAxWhcNMTcwODE3MDUxOTAxWjBfMQswCQYDVQQGEwJBVTETMBEGA1UECAwKU29t
            ZS1TdGF0ZTEhMB8GA1UECgwYSW50ZXJuZXQgV2lkZ2l0cyBQdHkgTHRkMRgwFgYD
            VQQDDA9mb28uZXhhbXBsZS5vcmcwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEK
            AoIBAQDZ9nd1isoGGeH4OFbQ6mpzlldo428LqEYSH4G7fhzLMKdYsIqkMRVl1J3s
            lXtsMQUUP3dcpnwFwKGzUvuImLHx8McycJKwOp96+5XD4QAoTKtbl59ZRFb3zIjk
            Owd94Wp1lWvptz+vFTZ1Hr+pEYZUFBkrvGtV9BoGRn87OrX/3JI9eThEpksr6bFz
            QvcGPrGXWShDJV/hTkWxwRicMMVZVSG6niPusYz2wucSsitPXIrqXPEBKL1J8Ipl
            8dirQLsH02ZZKcxGctEjlVgnpt6EI+VL6fs5P6A45oJqWmfym+uKztXBXCx+aP7b
            YUHwn+HV4qzZQld80PSTk6SS3hMXAgMBAAEwDQYJKoZIhvcNAQEFBQADggEBAKgf
            x3K9GHDK99vsWN8Ej10kwhMlBWBGuM0wkhY0fbxJ0gW3sflK8z42xMc2dhizoYsY
            sLfN0aylpN/omocl+XcYugLHnW2q8QdsavWYKXqUN0neIMr/V6d1zXqxbn/VKdGr
            CD4rJwewBattCIL4+S2z+PKr9oCrxjN4i3nujPhKv/yijhrtV+USw1VwuFqsYaqx
            iScC13F0nGIJiUVs9bbBwBKn1c6GWUHHiFCZY9VJ15SzilWAY/TULsRsHR53L+FY
            mGfQZBL1nwloDMJcgBFKKbG01tdmrpTTP3dTNL4u25+Ns4nrnorc9+Y/wtPYZ9fs
            7IVZsbStnhJrawX31DQ=
            -----END CERTIFICATE-----
        private:
          path: /usr/local/etc/ssl/bar/bar.key
          owner: "{% if ansible_os_family == 'FreeBSD' or ansible_os_family == 'OpenBSD' %}www{% elif ansible_os_family == 'RedHat' %}ftp{% else %}www-data{% endif %}"
          group: "{% if ansible_os_family == 'FreeBSD' or ansible_os_family == 'OpenBSD' %}www{% elif ansible_os_family == 'RedHat' %}ftp{% else %}www-data{% endif %}"
          key: |
            -----BEGIN RSA PRIVATE KEY-----
            MIIEowIBAAKCAQEA2fZ3dYrKBhnh+DhW0Opqc5ZXaONvC6hGEh+Bu34cyzCnWLCK
            pDEVZdSd7JV7bDEFFD93XKZ8BcChs1L7iJix8fDHMnCSsDqfevuVw+EAKEyrW5ef
            WURW98yI5DsHfeFqdZVr6bc/rxU2dR6/qRGGVBQZK7xrVfQaBkZ/Ozq1/9ySPXk4
            RKZLK+mxc0L3Bj6xl1koQyVf4U5FscEYnDDFWVUhup4j7rGM9sLnErIrT1yK6lzx
            ASi9SfCKZfHYq0C7B9NmWSnMRnLRI5VYJ6behCPlS+n7OT+gOOaCalpn8pvris7V
            wVwsfmj+22FB8J/h1eKs2UJXfND0k5Okkt4TFwIDAQABAoIBAHmXVOztj+X3amfe
            hg/ltZzlsb2BouEN7okNqoG9yLJRYgnH8o/GEfnMsozYlxG0BvFUtnGpLmbHH226
            TTfWdu5RM86fnjVRfsZMsy+ixUO2AaIG444Y4as7HuKzS2qd5ZXS1XB8GbrCSq7r
            iF/4tscQrzoG0poQorP9f9y60+z3R45OX3QMVZxP4ZzxXAulHGnECERjLHM5QzTX
            ALV9PHkTRNd1tm9FSJWWGNO5j4CGxFsPL1kdMyvrC7TkYiIiCQ/dd2CIfQyWwyKc
            8cHBKnzon0ugr0xlf2B0C7RTXrGAcuBC0yyaLuQTFkocUofgDIFghItH8O8xvvAG
            j8HYOwECgYEA9uMLtm2C8SiWFuafrF/pPWvhkBtEHA2g22M29CANrVv1jCEVMti/
            7r53fd328/nVxtashnSFz7a3l3s9d9pTR/rk/rNpVS2i7JGvCXXE3DeoD6Zf4utD
            MLEs2bI0KabdamIywc77CkVj9WUKd53tlcdcn7AsHwESU4Zjk08ie0kCgYEA4gIa
            R+a9jmKEk9l5Gn7jroxDJdI0gEfuA7It5hshEDcSvjF+Fs5+1tVgfBI1Mx4/0Eaj
            6E57Ln3WFKPJKuG0HwLNanZcqLFgiC/7ANbyKxfONPVrqC2TClImBhkQ74BLafZg
            yY8/N/g/5RIMpYvQ9snBRsah9G2cBfuPTHjku18CgYBHylPQk12dJJEoTZ2msSkQ
            jDtF/Te79JaO1PXY3S08+N2ZBtG0PGTrVoVGm3HBFif8rtXyLxXuBZKzQMnp/Rl0
            d9d43NDHTQLwSZidZpp88s4y5s1BHeom0Y5aK0CR0AzYb3+U7cv/+5eKdvwpNkos
            4JDleoQJ6/TZRt3TqxI6yQKBgA8sdPc+1psooh4LC8Zrnn2pjRiM9FloeuJkpBA+
            4glkqS17xSti0cE6si+iSVAVR9OD6p0+J6cHa8gW9vqaDK3IUmJDcBUjU4fRMNjt
            lXSvNHj5wTCZXrXirgraw/hQdL+4eucNZwEq+Z83hwHWUUFAammGDHmMol0Edqp7
            s1+hAoGBAKCGZpDqBHZ0gGLresidH5resn2DOvbnW1l6b3wgSDQnY8HZtTfAC9jH
            DZERGGX2hN9r7xahxZwnIguKQzBr6CTYBSWGvGYCHJKSLKn9Yb6OAJEN1epmXdlx
            kPF7nY8Cs8V8LYiuuDp9UMLRc90AmF87rqUrY5YP2zw6iNNvUBKs
            -----END RSA PRIVATE KEY-----
      - name: foobar-ca
        state: present
        ca:
          key: |
            -----BEGIN CERTIFICATE-----
            MIIDOjCCAiICCQDaGChPypIR9jANBgkqhkiG9w0BAQUFADBfMQswCQYDVQQGEwJB
            VTETMBEGA1UECAwKU29tZS1TdGF0ZTEhMB8GA1UECgwYSW50ZXJuZXQgV2lkZ2l0
            cyBQdHkgTHRkMRgwFgYDVQQDDA9mb28uZXhhbXBsZS5vcmcwHhcNMTcwNzE4MDUx
            OTAxWhcNMTcwODE3MDUxOTAxWjBfMQswCQYDVQQGEwJBVTETMBEGA1UECAwKU29t
            ZS1TdGF0ZTEhMB8GA1UECgwYSW50ZXJuZXQgV2lkZ2l0cyBQdHkgTHRkMRgwFgYD
            VQQDDA9mb28uZXhhbXBsZS5vcmcwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEK
            AoIBAQDZ9nd1isoGGeH4OFbQ6mpzlldo428LqEYSH4G7fhzLMKdYsIqkMRVl1J3s
            lXtsMQUUP3dcpnwFwKGzUvuImLHx8McycJKwOp96+5XD4QAoTKtbl59ZRFb3zIjk
            Owd94Wp1lWvptz+vFTZ1Hr+pEYZUFBkrvGtV9BoGRn87OrX/3JI9eThEpksr6bFz
            QvcGPrGXWShDJV/hTkWxwRicMMVZVSG6niPusYz2wucSsitPXIrqXPEBKL1J8Ipl
            8dirQLsH02ZZKcxGctEjlVgnpt6EI+VL6fs5P6A45oJqWmfym+uKztXBXCx+aP7b
            YUHwn+HV4qzZQld80PSTk6SS3hMXAgMBAAEwDQYJKoZIhvcNAQEFBQADggEBAKgf
            x3K9GHDK99vsWN8Ej10kwhMlBWBGuM0wkhY0fbxJ0gW3sflK8z42xMc2dhizoYsY
            sLfN0aylpN/omocl+XcYugLHnW2q8QdsavWYKXqUN0neIMr/V6d1zXqxbn/VKdGr
            CD4rJwewBattCIL4+S2z+PKr9oCrxjN4i3nujPhKv/yijhrtV+USw1VwuFqsYaqx
            iScC13F0nGIJiUVs9bbBwBKn1c6GWUHHiFCZY9VJ15SzilWAY/TULsRsHR53L+FY
            mGfQZBL1nwloDMJcgBFKKbG01tdmrpTTP3dTNL4u25+Ns4nrnorc9+Y/wtPYZ9fs
            7IVZsbStnhJrawX31DQ=
            -----END CERTIFICATE-----

  handlers:

    # If any certificate was changed, the nginx daemon will be restarted.
    - name: x509_certificate certificate changed > Restart nginx
      listen: x509_certificate certificate changed
      service:
        name: nginx
        state: restarted

    # If a certificate was changed, it will check if postfix use this certificate (when condition)
    # When it is true, it will notify the handler defined inside the postfix role
    - name: x509_certificate certificate changed > Check postfix and reload
      listen: x509_certificate certificate changed
      notify: Check postfix and reload
      debug:
        msg: x509_certificate certificate changed > Check postfix and reload
      changed_when: true
      no_log: true
      # The two variables postfix__tls_cert_file and postfix__tls_key_file contain the full path of the certificates used by postfix
      when: _x509_changed_certificates | selectattr('path', 'in', [postfix__tls_cert_file,postfix__tls_key_file]) | list
```

# License

```
Copyright (c) 2017 Tomoyuki Sakurai <y@trombik.org>

Permission to use, copy, modify, and distribute this software for any
purpose with or without fee is hereby granted, provided that the above
copyright notice and this permission notice appear in all copies.

THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF
MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR
ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN
ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF
OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.
```

```
Copyright (c) 2020 JoelKle <34544090+JoelKle@users.noreply.github.com>

Permission to use, copy, modify, and distribute this software for any
purpose with or without fee is hereby granted, provided that the above
copyright notice and this permission notice appear in all copies.

THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF
MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR
ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN
ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF
OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.
```

# Author Information

JoelKle <34544090+JoelKle@users.noreply.github.com>
