# install-on-CN

A guide to installing various programs on the CN machines, where
we do not have root/sudo permissions.

The guide assumes you have an SSH client and have already SSHed into the 
CN gateway (cn0) and from there, a computation CN machine that isn't too 
heavily loaded. If the software were built for cn0, it wouldn't work on 
the other CN machines (which are all 64-bit) and all the compiling would
cause unnecessary load on cn0.

In the few steps that involve editing a text file, this guide refers to 
the console-based editor, nano, which will not require X11 forwarding on 
the SSH connections. If you wish to use a graphical text editor, such 
as gedit, be sure to enable X11 (for *both* SSH connections) with the -X 
command line option, `ssh -X <USER>@<HOST>`.

## Explanation of folders

I tend to put small standalone executables right in `~/linux/bin/`.

For larger programs whose install consists of several files/folders,
e.g. Python, I put them in their own folder within `~/programs/` which 
is a folder I have created for the purpose of installing stuff. 
Then I create symlinks in `~/linux/bin/` which point to the relevant 
executable(s), or add the folder, e.g. `~/programs/name_of_tool/bin/` 
which contains those executables to the shell's PATH variable.

Assuming you don't already have a `~/programs` folder, create one for
the purpose of following along with the guide.

```bash
mkdir ~/programs
```

I keep enormous programs in `/data/<USERNAME>/programs-big` and create 
symlinks or add to the PATH as described above. There aren't a lot of 
these I have to deal with. For example though, if you build ANTs, a 
package for image registration, following the developer's recommended 
install method, it takes up 5.7 GB (including ITK, which is also 
downloaded and built in that process). This is a big step toward 
hitting the home directory size cap, and such programs probably don't
need to be backed up often, so the data directory is a good fit.


## Set up shell

This guide assumes you're using the tcsh shell, which the CN machines 
are configured to use by default. If you were wily enough to make it
launch bash instead of tcsh when you log in, you probably know where to
make the appropriate changes to files and commands listed below.

Now edit the `~/.cshrc_linux` file so the user level bin folder has
priority over other locations when the shell searches by name for an
executable. If the file doesn't already exist, the editor should create
it.

```bash
nano ~/.cshrc_linux
```

add the following line to the end of the file, then save it. 

```bash
setenv PATH ~/linux/bin:${PATH}
```

After copying text from the guide, it can be pasted into nano with
**Ctrl+Shift+V**. To save the file ("WriteOut"), **Ctrl+O** (the 
letter o), then press enter without changing the filename, and finally,
quit the editor with **Ctrl+X**.


#### Note:
Either log out and back in for this change to take effect, or paste 
that `setenv ` command into the shell and run it.


The way the CN machines are configured, their login scripts will add 
the `~/linux/bin` folder to the end of the PATH. This means that
`/usr/bin/python` will be found before `~/linux/bin/python`, so the 
wrong version is launched when `python` is typed in the terminal.

Now that this line has been added to `~/.cshrc_linux`, `~/linux/bin` 
will appear before other entries that the system adds to the path. So 
`~/linux/bin/python` will now be found before `/usr/bin/python`.



## Installing ack

Ack is like an enhanced version of grep with many features that are
helpful for programmers. http://beyondgrep.com/why-ack/

Since ack is contained in one file of modest size, it can go directly
in `~/linux/bin`

```bash
cd ~/linux/bin
curl http://beyondgrep.com/ack-2.12-single-file -o ack 
chmod 0755 ack
```

After adding a new executable to a folder in the PATH, the shell must be
notified or it may not see the executable. This can be done with

```bash
source ~/.cshrc_linux
```

### Ack usage

Installation is as simple as that. Ack is designed to only search source 
code files by default, so it won't waste any time chewing through huge
data files or binaries that happen to be in the search path. Specifying
options on the command line can place further restrictions on the
filetypes that ack will search.

As an example, say you are collaborating with someone and using their
Matlab scripts. You find that their scripts are working for them, but
failing for you due to tildes ( `~` ) in expressions that add folders to
the Matlab path during execution. They expand to 
`/export/home/<THEIR_USERNAME>/rest/of/path` for them, and expand to 
`/export/home/<YOUR_USERNAME>/rest/of/path` for you. The following ack
search can quickly identify many problem lines:

```bash
ack 'addpath ~' /path/to/their/matlab/files
```

And if there are a lot of other text files in that path, the ack 
search may be sped up by restricting it to only look in Matlab files

```bash
ack --matlab 'addpath ~' /path/to/their/matlab/files
```

By default, ack will descend into subdirectories of the search location.
It's great for searching in large codebases to find things with a known
name but unknown location (e.g. calls to a particular function). One of
my favorite command line options is `-C` for printing additional lines of context before and after each line where the search term was found.

```bash
ack -C 2 --cpp '->problem_function(' ~/code/large_c++_project_root_dir
```


## Installing Python

Download Python and build it from the source. The necessary build tools
are already available on the CN machines. There should be a 
`~/Downloads` folder that the system created when the user account was
set up, if not

```bash
mkdir ~/Downloads
```

then

```bash
cd ~/Downloads
curl https://www.python.org/ftp/python/2.7.7/Python-2.7.7.tgz -o Python-2.7.7.tgz
```

Expand the archive that was just downloaded (this will generate
a lot of text, showing the files as they're extracted).

```bash
tar xzvf Python-2.7.7.tgz
```

Move into the Python folder and configure it for building. This 
may take a few minutes.

```bash
cd Python-2.7.7
./configure
```

Here comes the crucial step for installing python at the user level,
rather than installing it system wide. This step may also take a few
minutes.

```bash
make altinstall prefix=~/programs/python exec-prefix=~/programs/python
```

Create a symlink in the user level bin folder to the python2.7 
executable, then refresh the shell

```bash
ln -s ~/programs/python/bin/python2.7 ~/linux/bin/python
source ~/.cshrc_linux
```

Check to make sure that everything so far is set up as expected.
If the command

```bash
which python
```

prints `/export/home/<YOUR_USERNAME>/linux/bin/python`, then the
configuration has been performed properly. If it prints 
`/usr/bin/python`, then one of the steps above may not have been
carried out correctly.



### Python usage

There is now a user level installation of Python. Feel free to launch
it and test it by playing around in the interactive shell. The prompt
will change to `>>> `, and calling the exit function, `>>> exit()`, 
will close the interactive shell.

Now, set an environment variable so Python package managers install
to the correct location.

```bash
nano ~/.cshrc_linux
```

and add this line to the end of the file.

```bash
setenv PYTHONHOME ~/programs/python
```

#### Note:
As before, after saving the change to this file, log out and back in,
or run the command in the terminal so the changes take effect.

## Installing pip, a Python package manager

How do you install a package manager if you don't already have the
package manager, which handles this sort of thing? Perform a
bootstrap setup of the Python package manager called easy_install.

```bash
cd ~/Downloads
curl https://bootstrap.pypa.io/ez_setup.py -o ez_setup.py
python ez_setup.py --insecure
```

By default, ez_setup uses wget for downloading additional files. On the
CN machines, wget has a tendency to fail, so tell ez_setup to use its
less-secure built in download functionality with the `--insecure` flag.

Now easy_install can be used to install pip, a very popular package 
manager for Python. Since it's only going to be called this one time,
instead of making an `easy_install` symlink in the bin folder, just
call it by specifying its location.

```bash
~/programs/python/bin/easy_install pip
```

Now there is a pip executable in `~/programs/python/bin`. The only thing
left is creating a symlink

```bash
ln -s ~/programs/python/bin/pip ~/linux/bin/pip
source ~/.cshrc_linux
```

## Cleanup

The downloaded files can be deleted with the following commands, 
which do not ask for confirmation, so be sure not to delete the 
wrong files.

```bash
cd ~/Downloads

rm -f ez_setup.py
rm -f setuptools-*.zip  # Downloaded by ez_setup.py

rm -f Python-2.7.7.tgz
rm -rf Python-2.7.7     # Extracted from the .tgz
```


## Using pip to install packages

A lot of steps were involved in getting Python and pip set up. 
Now, using pip, it is very easy to install additional packages for
Python.

As an example, search pip for packages with dicom in the name

```bash
pip search dicom
```

Install the desired one, pydicom

```bash
pip install pydicom
```

## Final thought

If you install a lot of very large Python packages, you may want to 
reconsider the location for the Python install. You could repeat
the steps here, altered slightly so that the install goes in 
`/data/<YOUR_USERNAME>/programs-big/python` instead.

Happy computing!
