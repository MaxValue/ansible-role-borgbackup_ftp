# Ansible Role: Borgbackup FTP

An Ansible Role that backups a directory of an FTP server to a borgbackup repository.

This role only runs on the ansible controller.

[TOC]

## Requirements

* [lftp](https://lftp.yar.ru/)
* optional: [borg](https://borgbackup.readthedocs.io/en/stable/installation.html)

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

### Main Variables

    ftp_vars: "set ssl:check-hostname false && set ftp:list-options -a &&"

The optional variable `ftp_vars` sets additional variables (settings) for lftp. Make sure to separate multiple options with the `&&` phrase and end the string with `&&` as well.

    ftp_host

The mandatory variable `ftp_host` sets the address of the FTP server.

    ftp_user

The optional variable `ftp_user` sets the name of the FTP user. If none is defined, the role tries to download the directory anonymously.

    ftp_password

The optional variable `ftp_password` sets the password of the FTP user.

    ftp_mirror_opts: "--delete --parallel=2"

The optional variable `ftp_mirror_opts` sets additional parameters for the `mirror` command of lftp.

    ftp_src

The mandatory variable `ftp_src` defines the FTP path which should be downloaded.

    ftp_dest

The optional variable `ftp_dest` defines the path where the contents of the FTP directory will be stored.

    ftp_speedup: true

The optional variable `ftp_speedup` determines whether the FTP download speed should be optimized.
If true, this adds the option `--only-newer` to `ftp_mirror_opts` (if not customized)
and tries to checkout the latest borg archive from the given repo before downloading the directory on top of it.

    ftp_download_cleanup: true

The optional variable `ftp_download_cleanup` determines whether the downloaded directory should be deleted when everything has been completed.

    borgbackup_perform: true

The optional variable `borgbackup_perform` determines whether a borg backup of the downloaded directory will be performed or not.

    borgbackup_borgpath: "borg"

The optional variable `borgbackup_borgpath` sets the path to the borg executable.

    borgbackup_sshcommand

The optional variable `borgbackup_sshcommand` sets the command used to connect via SSH to the repository. This can be used to supply additional options to the SSH command.

    borgbackup_repo

The mandatory variable `borgbackup_repo` sets the URL to the borg repository.

    borgbackup_passphrase

The mandatory variable `borgbackup_passphrase` sets the passphrase to interact with the borg repository.

    borgbackup_archivenameformat: "{{ ftp_host }}-{{ _borgbackup_timeformat }}"

The optional variable `borgbackup_archivenameformat` sets the name format used for naming the borg archives.

    borgbackup_archivenameprefix

The optional variable `borgbackup_archivenameprefix` defines a prefix in front of borg archive names.
This feature is untested.

    borgbackupftp_nolog: true

The optional variable `borgbackupftp_nolog` determines whether secrets should be logged or not.

### Background variables

These variables define specific defaults which can be overriden, if wanted.

    _borgbackup_timeformat: "%Y-%m-%dT%H:%M:%S"

The optional variable `_borgbackup_timeformat` time format used by the borg `--timestamp` parameter (a [strptime](https://docs.python.org/3/library/datetime.html#strftime-and-strptime-behavior) string). This probably never needs to change.

    borgbackup_timeout: [null, 600]

The optional variable `[null, 600]` defines timeouts for the `borg create` command via a 2 item array.
The first item defines the timeout when no archive already exists (usually the first archive takes very long)
and the second item defines the timeout when at least one archive already exists.

    borgbackup_timeout_prepare: null

The optional variable `borgbackup_timeout_prepare` defines the timeout when checking out the last borg archive.
Of course, this is only relevant when `borgbackup_perform` and `ftp_speedup` are `true`.

### Internal variables

These are variables internally created by the role. Please do not use them in your code.

    _result_tmpdir_dest

The internal variable `_result_tmpdir_dest` stores the path to the generated tmp directory for storing the downloaded FTP contents.

    _result_ftp_download

The internal variable `_result_ftp_download` stores the task result of the lftp download task.
This is needed in order to determine if files were actually downloaded which influences the `changed` state of the task.

    _result_borglist_raw

The internal variable `_result_borglist_raw` contains the string response from the `borg list` command which is used to determine the latest archive in the borg repository.

    _result_borglist

The internal variable `_result_borglist` contains the JSON response from the `borg list` command which is used to determine the latest archive in the borg repository.

    _borgbackup_timeout

The internal variable `_borgbackup_timeout` contains the actual timeout value which is derived from `_result_borglist` and `borgbackup_timeout`.

    _nullval

The internal varialbe `_nullval` stores the value `null` because Ansible would complain if used directly.

## Dependencies

* The community.general collection: `ansible-galaxy collection install --force community.general`

## Example Playbooks

### Normal usage

    ---
    # This downloads an FTP directory and creates a borg archive of it
    - hosts: localhost
      roles:
        - role:
            name: maxvalue.borgbackup_ftp
          vars:
            ftp_host: MY.AWESOME.FTP.HOST.EXAMPLE.COM
            ftp_user: THATSME
            ftp_password: HEREISMYPRETTYFTPPASSWORD
            ftp_src: /my/remote/ftp/dir
            borgbackup_repo: borguser@borghost:22/path/to/borg/repo
            borgbackup_sshcommand: ssh -i /home/myname/.ssh/borgbackup_ed25519
            borgbackup_passphrase: HEREISMYBORGPASSPHRASE
    ...

### Only download an FTP directory

    ---
    # This downloads an FTP directory
    - hosts: localhost
      roles:
        - role:
            name: maxvalue.borgbackup_ftp
          vars:
            ftp_host: MY.AWESOME.FTP.HOST.EXAMPLE.COM
            ftp_user: THATSME
            ftp_dest: /the/path/where/to/download/to
            borgbackup_perform: false
    ...

## License

[MIT](LICENSE.txt)

## Sponsors

You can support the development of this role and other similar roles by donating to one of the accounts below.

* [bymeacoffee](https://www.buymeacoffee.com/publicbetamax)
* [liberapay](https://de.liberapay.com/maxvalue/)
* [ko-fi](https://ko-fi.com/publicbetamax)
* [Patreon](patreon.com/publicbetamax)

## Author Information

This role was created in 2022 by Max Fuxjäger:
* Mastodon: @maxvalue@chaos.social
* Matrix: @ypsilon:matrix.org
