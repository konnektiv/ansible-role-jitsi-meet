jitsi-meet
=========

Installs and configures the [Jitsi Meet] videoconferencing software.


Requirements
------------

You should have DNS pointed at the server already, and SSL keys. If you don't have SSL
keys for the domain yet, consider using the excellent [thefinn93.letsencrypt] Ansible role
to obtain (free!) SSL certs from [LetsEncrypt].

You will also need to expose ports 443 TCP and 10000 UDP for the Jitsi Meet
components to work. By default the role will use `ufw` to allow these ports. If you
use another host-based firewall solution such as iptables, set
`jitsi_meet_configure_firewall: false`. If you use AWS or similar, you'll need to
expose those ports in the associated Security Group.

Role Variables
--------------

```yaml

# Without SSL, "localhost" is the correct default. If SSL info is provided,
# then we'll need a real domain name. Using Ansible's inferred FQDN, but you
# can set the variable value explicitly if you use a shorter hostname
# If automatic Nginx configuration is disabled, also use FQDN, since presumably
# another role will manage the vhost config.
jitsi_meet_server_name: "{{ ansible_fqdn }}"

# "anonymous" and "internal_plain" is supported
# when using "internal_plain", this configures jitsi meet as explained here:
# https://github.com/jitsi/jicofo#secure-domain
# you have to create users with 
# `prosodyctl register <username> jitsi-meet.example.com <password>`
jitsi_meet_authentication: anonymous

# A recent privacy-friendly addition, see here for details:
# https://github.com/jitsi/jitsi-meet/issues/422
# https://github.com/jitsi/jitsi-meet/pull/427
jitsi_meet_disable_third_party_requests: true
# These debconf settings represent answers to interactive prompts during installation
# of the jitsi-meet deb package. If you use custom SSL certs, you may have to set more options.
jitsi_meet_debconf_settings:
  - name: jitsi-meet
    question: jitsi-meet/cert-choice
    value: "{{ jitsi_meet_cert_choice }}"
    vtype: string
  - name: jitsi-meet-prosody
    question: jitsi-meet-prosody/jvb-hostname
    value: "{{ jitsi_meet_server_name }}"
    vtype: string
  - name: jitsi-videobridge
    question: jitsi-videobridge/jvb-hostname
    value: "{{ jitsi_meet_server_name }}"
    vtype: string

# webserver to use for jitsi meet. Either 'nginx' or 'apache2'
jitsi_meet_webserver: nginx

# UI customization
jitsi_meet_customize_the_ui: false

jitsi_meet_lang: 'en'
jitsi_meet_appname: 'My app name'
jitsi_meet_org_link: 'https://link-to-my-organization.com'

jitsi_meet_default_background: '#474747'
jitsi_meet_disable_video_background: 'false'
jitsi_meet_default_remote_display_name: 'Fellow Jitster'
jitsi_meet_default_local_display_name: 'me'
jitsi_meet_generate_roomnames_on_welcome_page: 'true'
jitsi_meet_lang_detection: 'false'    # Allow i18n to detect the system language

jitsi_meet_favicon_file: images/favicon.ico
jitsi_meet_logo_file: images/jitsilogo.png
jitsi_meet_watermark_file: images/watermark.png

jitsi_meet_customizations:
  - key: APP_NAME
    value: "'{{jitsi_meet_appname}}'"
  - key: NATIVE_APP_NAME
    value: "'{{jitsi_meet_appname}}'"
  - key: DEFAULT_REMOTE_DISPLAY_NAME
    value: "'{{jitsi_meet_default_remote_display_name}}'"
  - key: JITSI_WATERMARK_LINK
    value: "'{{jitsi_meet_org_link}}'"
  - key: DEFAULT_BACKGROUND
    value: "'{{jitsi_meet_default_background}}'"

jitsi_meet_configurations:
  - key: startAudioMuted
    value: 10
  - key: startWithAudioMuted
    value: 'false'
  - key: startVideoMuted
    value: 10
  - key: startWithVideoMuted
    value: 'false'
  - key: disableThirdPartyRequests
    value: "{{jitsi_meet_disable_third_party_requests}}"

# Role will automatically install configure ufw with jitsi-meet port holes.
# If you're managing a firewall elsewise, set this to false, and ufw will be skipped.
jitsi_meet_configure_firewall: false

```

Screen sharing
--------------
Scren sharing should just work with Chrome or Firefox without any extensions.

Dependencies
------------

It's technically not a dependency, but you should check out [thefinn93.letsencrypt]
for astoundingly easy SSL certs.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```yaml
- name: Configure jitsi-meet server.
  hosts: jitsi
  vars:
    # Change this to match the DNS entry for your host IP.
    jitsi_meet_server_name: meet.example.com
  roles:
    - role: thefinn93.letsencrypt
      become: yes
      letsencrypt_email: "webmaster@{{ jitsi_meet_server_name }}"
      letsencrypt_cert_domains:
        - "{{ jitsi_meet_server_name }}"
      tags: letsencrypt

    - role: ansible-role-jitsi-meet
      jitsi_meet_ssl_cert_path: "/etc/letsencrypt/live/{{ jitsi_meet_server_name }}/fullchain.pem"
      jitsi_meet_ssl_key_path: "/etc/letsencrypt/live/{{ jitsi_meet_server_name }}/privkey.pem"
      become: yes
      tags: jitsi
```

Running the tests
-----------------

This role uses [Molecule] and [ServerSpec] for testing. To use it:

```
pip install molecule
gem install serverspec
molecule test
```

You can also run selective commands:

```
molecule idempotence
molecule verify
```

See the [Molecule] docs for more info.

License
-------

MIT

Author Information
------------------
[Konnektiv] 

[Konnektiv]: https://konnektiv.de 
[Jitsi Meet]: https://github.com/jitsi/jitsi-meet
[thefinn93.letsencrypt]: https://github.com/thefinn93/ansible-letsencrypt
[LetsEncrypt]: https://letsencrypt.org/
[Freedom of the Press Foundation]: https://freedom.press/
[Molecule]: http://molecule.readthedocs.org/en/master/
[ServerSpec]: http://serverspec.org/
[Jidesha]: https://github.com/jitsi/jidesha
