==========
User Guide
==========

Command line syntax
===================

Use the ``--help`` or ``-h`` argument to get help::

    $ pyftpsync -h
    usage: pyftpsync [-h] [--verbose | --quiet] [--version] [--progress]
                     {upload,download,sync} ...

    Synchronize folders over FTP.

    positional arguments:
      {upload,download,sync}
                            sub-command help
        upload              copy new and modified files to remote folder
        download            copy new and modified files from remote folder to
                            local target
        sync                synchronize new and modified files between remote
                            folder and local target

    optional arguments:
      -h, --help            show this help message and exit
      --verbose, -v         increment verbosity by one (default: 3, range: 0..5)
      --quiet, -q           decrement verbosity by one
      --version             show program's version number and exit
      --progress, -p        show progress info, even if redirected or verbose < 3
   $


Upload files syntax
-------------------

Command specific help is available like so::

    $ pyftpsync upload --help
    usage: pyftpsync upload [-h] [-x] [-f INCLUDE_FILES] [-o OMIT]
                            [--store-password] [--no-prompt] [--no-color]
                            [--force] [--resolve {local,skip,ask}] [--delete]
                            [--delete-unmatched]
                            LOCAL REMOTE

    positional arguments:
      LOCAL                 path to local folder (default: .)
      REMOTE                path to remote folder

    optional arguments:
      -h, --help            show this help message and exit
      -x, --execute         turn off the dry-run mode (which is ON by default),
                            that would just print status messages but does not
                            change anything
      -f INCLUDE_FILES, --include-files INCLUDE_FILES
                            wildcard for file names (default: all, separate
                            multiple values with ',')
      -o OMIT, --omit OMIT  wildcard of files and directories to exclude (applied
                            after --include)
      --store-password      save password to keyring if login succeeds
      --no-prompt           prevent prompting for missing credentials
      --no-color            prevent use of ansi terminal color codes
      --force               overwrite remote files, even if the target is newer
                            (but no conflict was detected)
      --resolve {local,skip,ask}
                            conflict resolving strategy (default: 'skip')
      --delete              remove remote files if they don't exist locally
      --delete-unmatched    remove remote files if they don't exist locally or
                            don't match the current filter (implies '--delete'
                            option)
   $

Example: Upload files
---------------------

Upload all new and modified files from user's temp folder to an FTP server.
No files are changed on the local directory::

  $ pyftpsync upload ~/temp ftp://example.com/target/folder

Add the ``--delete`` option to remove all files from the remote target that
don't exist locally::

  $ pyftpsync upload ~/temp ftp://example.com/target/folder --delete

Add the ``-x`` option to switch from DRY-RUN mode to real execution::

  $ pyftpsync upload ~/temp ftp://example.com/target/folder --delete -x

Replace ``ftp://`` with ``ftps://`` to enable TLS encryption.


Synchronize files syntax
------------------------
::

    $ pyftpsync sync --help
    usage: pyftpsync sync [-h] [-x] [-f INCLUDE_FILES] [-o OMIT]
                          [--store-password] [--no-prompt] [--no-color]
                          [--resolve {old,new,local,remote,skip,ask}]
                          LOCAL REMOTE

    positional arguments:
      LOCAL                 path to local folder (default: .)
      REMOTE                path to remote folder

    optional arguments:
      -h, --help            show this help message and exit
      -x, --execute         turn off the dry-run mode (which is ON by default),
                            that would just print status messages but does not
                            change anything
      -f INCLUDE_FILES, --include-files INCLUDE_FILES
                            wildcard for file names (default: all, separate
                            multiple values with ',')
      -o OMIT, --omit OMIT  wildcard of files and directories to exclude (applied
                            after --include)
      --store-password      save password to keyring if login succeeds
      --no-prompt           prevent prompting for missing credentials
      --no-color            prevent use of ansi terminal color codes
      --resolve {old,new,local,remote,skip,ask}
                            conflict resolving strategy (default: 'ask')
    $

Example: Synchronize folders
----------------------------

Two-way synchronization of a local folder with an FTP server::

  $ pyftpsync sync --store-password --resolve=ask --execute ~/temp ftps://example.com/target/folder

Note that ``ftps:`` protocol was specified to enable TLS.


Script examples
===============

Upload changes from local folder to FTP server::

  from ftpsync.targets import FsTarget
  from ftpsync.ftp_target import FtpTarget
  from ftpsync.synchronizers import UploadSynchronizer

  local = FsTarget("~/temp")
  user ="joe"
  passwd = "secret"
  remote = FtpTarget("/temp", "example.com", username=user, password=passwd)
  opts = {"force": False, "delete_unmatched": True, "verbose": 3, "dry_run" : False}
  s = UploadSynchronizer(local, remote, opts)
  s.run()

Synchronize local folder with FTP server using TLS::

  from ftpsync.targets import FsTarget
  from ftpsync.ftp_target import FtpTarget
  from ftpsync.synchronizers import BiDirSynchronizer

  local = FsTarget("~/temp")
  user ="joe"
  passwd = "secret"
  remote = FtpTarget("/temp", "example.com", username=user, password=passwd, tls=True)
  opts = {"resolve": "skip", "verbose": 1, "dry_run" : False}
  s = BiDirSynchronizer(local, remote, opts)
  s.run()
