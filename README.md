![Public Domain](src/var/www/htdocs/mercury.example.com/Milonia_Caesonia-250x259.jpg)

Ansible role for a Caesonia (beta)
=================================

Ansible role to create a Mailserver on OpenBSD (>6.2) with OpenSMTPD, Dovecot, DKIMProxy and Rspamd.

Requirements
------------

OpenBSD, Python 2.7 (on client machine) and 10 minutes.

Notes
-----

This is still a WIP, so far, you need to create DKIM keys, new users and DNS entrys. Also, you need
to enable dovecot, smtpd, rspamd and dkimproxy_{in,out} at boot.

You need to adjust your pf.conf (example bellow).

Also, you need to delete examples on /etc/mail/virtuals and /etc/mail/domains

Feedback is welcome.

Example pf.conf
---------------

For IMAPs I like to keep bruteforce people away with some tweaks on pf.conf, keep in mind that
lower numbers than that, can couse problems checking emails on huge directories like misc or mailing list.

```
...
pass in quick on egress proto tcp from any \
        to (egress) port imaps \
        flags S/SA modulate state \
        (max-src-conn 50, max-src-conn-rate 50/5, overload <bruteforce> flush global)
...
```

For SMTP I have pretty the same:

```
...
pass in quick log (to pflog1) proto tcp from any \
        to (egress) port smtp

pass in quick log (to pflog1) proto tcp from any \
        to (egress) port { submission, smtps } \
        flags S/SA modulate state \
        (max-src-conn 50, max-src-conn-rate 25/5, overload <bruteforce> flush global)
...
```

Example Playbook
----------------
```
---
- hosts: test
  roles:
     - role: vedetta-com.caesonia
  become: yes
  become_method: doas

  vars:
   domain: 'foobar.com'
   mail_dir: '/var/vmail'
   mail_user: 'gonzalo'
   release: '6.2'
   arch: 'amd64'
   installurl_mirror: 'https://fastly.cdn.openbsd.org/pub/OpenBSD/'
   pkg_path: 'https://fastly.cdn.openbsd.org/pub/OpenBSD/{{ release }}/packages/{{ arch }}/'
   packages_list:
    - dovecot
    - dovecot-pigeonhole
    - dkimproxy
    - rspamd
    - opensmtpd-extras
```

License
-------

BSD

