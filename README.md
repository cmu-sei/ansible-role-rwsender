rwsender
=========

A role for configuring and managing the rwsender service.  rwsender is a daemon which watches an incoming directory for files, moves the files into a processing directory, and transfers the files to one or more [rwreceiver](https://tools.netsa.cert.org/silk/docs.html#rwreceiver) processes. See rwsender [documentation](https://tools.netsa.cert.org/silk/rwsender.html) for more information.

Requirements
------------

If using TLS for connections, matching certs need to be generated and uploaded to both sender and receiver.

Role Variables
--------------

Available variables are listed below, along with default values (see [defaults/main.yml](defaults/main.yml)):

	silk_packing_tools_loc: "/usr/local/sbin"

The folder location of the silk packing tools.

	silk_tls_support: False

Whether to use TLS for connections.

	rwsender_myname: "rwsender"

The name of the rwsender process.  It is possible to run multiple copies of rwsender on a single box by naming them differently.

	rwsender_conf_template: "rwsender.conf.j2"
	rwsender_conf_file_loc: "/usr/local/etc"
	rwsender_conf_file_path: "{{ rwsender_conf_file_loc }}/{{ rwsender_myname }}.conf"
	rwsender_init_template: "rwsender.j2"
	rwsender_init_file_path: "/etc/init.d/{{ rwsender_myname }}" 

Template sources to use and their destinations.

| Variable  | Explanation |
| ------------- | ------------- |
| rwsender_statedirectory: "/usr/local/var/lib/rwsender" | Directory for where rwsender keeps its state |
| rwsender_create_directories: "no" | If set to "yes", the defined directories will be created automatically if they do not already exist |
| rwsender_bin_dir: "{{ silk_packing_tools_loc }}" | Full path of the directory containing the "rwsender" program |
| rwsender_destination_dir: "{{ rwsender_statedirectory }}/destination" | The full path of the directory where received files will be placed |
| rwsender_mode: "client" | The mode in which the sender will run.  Valid values are "server" and "client". |
| rwsender_id: "sender-1" | The name of this sender instance |
| rwsender_port: "" | The PORT or HOST:PORT pair upon which the server listens for incoming connections.  This is only required when running in server mode.  If HOST is not provided, the server listens on any address. HOST may be a name or an IP address. If HOST is an IPv6 address, enclose it in square brackets, and enclose the entire value in single quotes to prevent interpretation by the shell. |
| rwsender_post_command: "" | Command to run on each file once it has been received.  In the command, "%s" will be replaced with the full path to the received file, and "%I" will be replaced with the identifier of the rwsender that sent the file. E.g.: POST_COMMAND='echo received file %s from rwsender %I' |
| rwsender_freespace_min: "0" | The amount of space (in bytes) rwsender will attempt to leave free in the file system containing $DESTINATION_DIR.  This variable may be set as an ordinary integer, or as a real number followed by a suffix K, M, G, or T. |
| rwsender_space_max_percent: "100" | The maximum percentage of space of the file system containing $DESTINATION_DIR that rwsender will be willing to use. |
| rwsender_sender_servers: "" | If the sender is in client mode, then SENDER_SERVERS must be specified.  The lines should have the following format: `<identifier> <host>:<port>`. These variables are multi-line values, where each line has the value for one rwsender. |
| rwsender_sender_clients: "" | If the sender is in server mode, then SENDER_CLIENTS must be specified.  The lines should have the following format: `<identifier>`. These variables are multi-line values, where each line has the value for one rwsender. |
| rwsender_duplicate_dirs: "" | The sender can copy incoming files to multiple destination directories.  Note that if copying to one of the following directories fails, an error is logged but the file is treated as having been successfully transferred. This variable contains a multi-line value, where each line lists a duplicate destination directory.  The format for the line is: `<duplicate-dir>`. Note that each line must be a complete directory path |
| rwsender_duplicate_copies: "link" | When copying files to a duplicate destination directory, by default rwsender creates the files as hard links (if possible) to each other and to files in the destination directory.  However, if some process modifies-in-place a copy in one location that will affect all files.  To force rwsender to create a complete copy, change DUPLICATE_COPIES from "link" to "copy". |
| rwsender_log_type: "syslog" | The type of logging to use.  Valid values are "legacy" and "syslog". |
| rwsender_log_level: "info" | The lowest level of logging to actually log.  Valid values are: emerg, alert, crit, err, warning, notice, info, debug |
| rwsender_log_dir: "{{ rwsender_statedirectory }}/log" | The full path of the directory where the log files will be written when LOG_TYPE is "legacy". |
| rwsender_pid_dir: "{{ rwsender_log_dir }}" | The full path of the directory where the PID file will be written |
| rwsender_user: "root" | The user this program runs as; root permission is required only when rwsender listens on a privileged port. |
| rwsender_extra_options: "" | Extra options to pass to rwsender |

When using the optional GnuTLS support by setting `silk_tls_support: True`, the full path the CA file (TLS_CA) must be specified, as well as **EITHER** the full path to the program-specific PKCS#12 file (TLS_PKCS12) *OR* the full paths to the program-specific certificate (TLS_CERT) and key (TLS_KEY) files. If the PKCS#12 file is password protected, you must set the RWsender_TLS_PASSWORD environment variable to the password prior to starting rwsender.  If RWsender_TLS_PASSWORD is not set, it is treated as a null password; set it to the empty string to allow for an empty password.


| TLS Variable  | Explanation |
| ------------- | ------------- |
| rwsender_tls_ca: "" | The full path to the root CA file, PEM encoded |
| rwsender_tls_pkcs: "" | The full path to the program specific PKCS#12 file, DER encoded |
| rwsender_tls_key: "" | The full path to the program specific key file, PEM encoded |
| rwsender_tls_cert: "" | The full path to the program specific certificate file, PEM encoded |
| rwsender_tls_crl: "" | The full path to the Certificate Revocation List, PEM encoded. Optional. |
| rwsender_tls_priority: "" | The preference order (priority) for ciphers, key exchange methods, message authentication codes, and compression methods.  Optional. The default is "NORMAL".  Available arguments depend on your version of GnuTLS. |
| rwsender_tls_security: "" | The security level to use when generating Diffie-Hellman parameters. One of low, medium, high, or ultra.  Optional.  The default is "medium". |
| rwsender_tls_debug_level: "" | The debugging level used internally by the GnuTLS library, a number between 0 (no debugging) and 99 inclusive.  Optional. |



Dependencies
------------

See [meta/main.yml](meta/main.yml).

Example Playbook
----------------

    - hosts: server
      become: true
      vars:
        data_root_dir: "/data"
        # rwsender settings
        silk_tls_support: True
        rwsender_statedirectory: "{{ data_root_dir }}/rwsender"
        rwsender_destination_dir: "{{ rwsender_statedirectory }}/incoming"
        rwsender_create_directories: "yes"
        rwsender_mode: "server"
        rwsender_port: "3737"
        rwsender_sender_clients: |
            sender1
            sender2
        # rwsender tls settings
        tls_ca: "testcert.pem"
        tls_key: "client-key.pem"
        tls_cert: "client-cert.pem"
        rwsender_tls_ca: "/etc/pki/tls/{{ tls_ca }}"
        rwsender_tls_key: "/etc/pki/tls/private/{{ tls_key }}"
        rwsender_tls_cert: "/etc/pki/tls/{{ tls_cert }}"
        rwsender_pid_dir: "/var/run"
      pre_tasks:
        - name: Copy ssl certs
          copy:
            src: "{{ item.f }}"
            dest: "{{ item.d }}"
            mode: "{{ item.m }}"
            owner: "root"
            group: "root"
          with_items:
            - f: "{{ tls_ca }}"
              d: "{{ rwsender_tls_ca }}"
              m: '0644'
            - f: "{{ tls_key }}"
              d: "{{ rwsender_tls_key }}"
              m: '0600'
            - f: "{{ tls_cert}}"
              d: "{{ rwsender_tls_cert }}"
              m: '0644'
      roles:
        - role: cmusei.rwsender
          tags: [ 'rwsender' ]

License
-------

Copyright 2020 Carnegie Mellon University.
NO WARRANTY. THIS CARNEGIE MELLON UNIVERSITY AND SOFTWARE ENGINEERING INSTITUTE MATERIAL IS FURNISHED ON AN "AS-IS" BASIS. CARNEGIE MELLON UNIVERSITY MAKES NO WARRANTIES OF ANY KIND, EITHER EXPRESSED OR IMPLIED, AS TO ANY MATTER INCLUDING, BUT NOT LIMITED TO, WARRANTY OF FITNESS FOR PURPOSE OR MERCHANTABILITY, EXCLUSIVITY, OR RESULTS OBTAINED FROM USE OF THE MATERIAL. CARNEGIE MELLON UNIVERSITY DOES NOT MAKE ANY WARRANTY OF ANY KIND WITH RESPECT TO FREEDOM FROM PATENT, TRADEMARK, OR COPYRIGHT INFRINGEMENT.
Released under a MIT (SEI)-style license, please see license.txt or contact permission@sei.cmu.edu for full terms.
[DISTRIBUTION STATEMENT A] This material has been approved for public release and unlimited distribution.  Please see Copyright notice for non-US Government use and distribution.
CERT® is registered in the U.S. Patent and Trademark Office by Carnegie Mellon University.
This Software includes and/or makes use of the following Third-Party Software subject to its own license:
1. ansible (https://github.com/ansible/ansible/tree/devel/licenses) Copyright 2019 Red Hat, Inc.
2. molecule (https://github.com/ansible-community/molecule/blob/master/LICENSE) Copyright 2018 Red Hat, Inc.
3. testinfra (https://github.com/philpep/testinfra/blob/master/LICENSE) Copyright 2020 Philippe Pepiot.

DM20-0508


Author Information
------------------

This role was created in 2019 by [Matt Heckathorn](https://resources.sei.cmu.edu/library/author.cfm?authorID=2403).
