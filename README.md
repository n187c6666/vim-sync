vim-sync
========

Automatic sync local and remote file in vim


Installation
----

Install using [bundle],[vundle],[pathogen] or your favorite Vim package manager.

Requires
---

[Asyncrun] to upload asynchronously.

Usage
----

create a execute file called .sync in your project directory.
e.g. /project_dir/ is your project dirctory and the execute file is /project_dir/.sync. how to write execute file see "some execute config" section.

    <leader>su
    Upload current buffer file, it will execute the command: project_dir/.sync upload current_buffer_fold current_file_name

    <leader>sd
    Download current buffer file, it will execute the command: project_dir/.sync download current_buffer_fold current_file_name

some execute config
----
* rsync:
<pre>
#!/bin/sh
if [ "upload" == $1 ];then
    rsync -azcuv -e "/bin/ssh -p36000 -q" `dirname $0`/$2/$3 login_name@remote_host:/remote_path/$2/$3
elif [ 'download' == $1 ];then
    rsync -azcuv -e "/bin/ssh -p36000 -q" login_name@remote_host:/remote_path/$2/$3 `dirname $0`/$2/$3
fi
</pre>
* sftp:
<pre>
#!/bin/sh
if [ "upload" == $1 ];then
    expect -c <<'END_EXPECT'
	set timeout -1
	spawn sftp login_name@1.2.3.4
	expect "[Pp]assword:"
	send "login_password\r"
	expect "sftp>"
	send "put `dirname $0`/$2/$3 /remote_path/$2/$3\r"
	expect "%100"
	send "quit\r"
	expect eof
	END_EXPECT
elif [ 'download' == $1 ];then
    expect -c <<'END_EXPECT'
	set timeout -1
	spawn sftp login_name@1.2.3.4
	expect "[Pp]assword:"
	send "login_password\r"
	expect "sftp>"
	send "get /remote_path/$2/$3 `dirname $0`/$2/$3 \r"
	expect "%100"
	send "quit\r"
	expect eof
	END_EXPECT
fi
</pre>
* ftp:
<pre>
#!/bin/sh
if [ "upload" == $1 ];then
  ncftpput -m -u login_name -p login_password -P 21 remote_host remote_path/$2 `dirname $0`/$2/$3
elif [ 'download' == $1 ];then
  ncftpget -u login_name -p login_password -P 21 remote_host `dirname $0`/$2 remote_path/$2/$3
fi
</pre>

* scp:

    referred to rsync
* ...

Configuration
----

* g:sync_exe_filenames (default: '.sync;')

Defines the filenames of the executable file to use to synchronize your sources.

This file will be searched from the directory of the file to synchronized.
For example, when editing a symlink and g:sync_push_symlink_too is enabled the target will first be synchronized and the file will be searched from the target directory.
Then the symlink will be synchronized and the file will be searched from the symlink directory.

You can prodive multiple filenames, separate with a ",".

To look backward in the directory tree add ";" at the end of the filname.

Example :

`let g:sync_exe_filenames = '.sync;' " Looks backward for a file named ".sync"`

`let g:sync_exe_filenames = '.sync;,.sync.sh; " Looks backward for a file named ".sync". If not found then looks backward for a file named ".sync.sh"'`

* g:sync_push_symlink_too (default: 0)

When editing a symlink, allows to synchronized the symlink itself.
If disabled, only the target is synchronized.

Might be usefull is you use a lot of symlinks and don't want to have
to push them manually to the remote.


* g:sync_async_upload (default: 1)

Defines if the upload should be asynchronous.
Requires the [Asyncryn] plugin.

Alias
----

If you want to another command, write following like.

Ctrl+u
    `nnoremap <C-U> <ESC>:call SyncUploadFile()<CR>`

Ctrl+d
    `nnoremap <C-D> <ESC>:call SyncDownloadFile()<CR>`

* if you want to auto upload/download file where save/open file, write these code in you .emacs config file:

        autocmd BufWritePost * :call SyncUploadFile()
        autocmd BufReadPre * :call SyncDownloadFile()


[bundle]:https://github.com/bundler/bundler/
[vundle]:https://github.com/gmarik/vundle/
[pathogen]:https://github.com/tpope/vim-pathogen/
[Asyncrun]:https://github.com/skywind3000/asyncrun.vim
