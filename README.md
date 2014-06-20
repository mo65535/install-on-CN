# install-on-CN

A guide to installing various programs on the CN machines, where
we do not have root/sudo permissions.


## Explanation of folders

I tend to put standalone executables that aren't too large, e.g. the
grep replacement, ack, directly in `~/linux/bin/`

For larger programs whose install consists of several files/folders,
e.g. Python, I put them in their own folder within `~/programs/` which 
is a folder I have created for the purpose of installing stuff. 
Then I create symlinks in `~/linux/bin/` which point to the relevant 
executable(s), or add the folder, e.g. `~/programs/name_of_tool/bin/` 
which contains those executables to my shell's PATH variable.

If you don't already have such a folder, create one now for following
along with the guide.

```bash
mkdir ~/programs
```

I keep enormous programs in `/data/<USERNAME>/programs-big` and create
symlinks or add to the PATH as described above.
There aren't a lot of these I have to deal with. For example though, 
if you build ANTs, a package for image registration, following the
developer's recommended install method, it takes up 5.7 GB (including 
ITK, which is also downloaded and built in that process).
This is a big step toward exhausting your home directory size cap, and
it probably doesn't need to be backed up often **(smaller scripts you
write to invoke ANTs should be, though)** so the data directory is a 
good fit.


## Set up shell

This section, and the rest of the guide, assumes you're using the 
tcsh shell which the CN machines are configured to use by default.
If you were wily enough to make it launch bash instead of tcsh when 
you log in, you probably know where to make the appropriate changes to
files and commands listed below.

If you don't already have one, create a `~/.cshrc_linux` file and add 
this line to it, then save it.

```bash
setenv PATH ~/linux/bin:${PATH}
```

**Note:** you can either log out and back in for this change to take 
effect, or you can paste this line into your shell and run it.

The way the CN machines are configured, their login scripts will add 
your `~/linux/bin` folder to your PATH at the end. This means that, for
instance, `/usr/bin/python` will be found before `~/linux/bin/python`,
so the wrong version is launched when you type `python` in the terminal.

After adding this line to `~/.cshrc_linux`, `~/linux/bin` will appear 
before other entries that the system adds to your path. It still adds
`~/linux/bin` at the end, too, but this duplication should not cause 
problems.



## Installing ack

Why might you want to use ack? Here is the creators' argument for why:

http://beyondgrep.com/why-ack/

Since ack is contained in one file of modest size, I put it directly
in `~/linux/bin`

```bash
cd ~/linux/bin
curl http://beyondgrep.com/ack-2.12-single-file -o ack 
chmod 0755 ack
```

After adding a new executable to a folder on your path, you may need to
refresh the shell so it finds the proper executable. Our `~/.cshrc_linux`
file adds `~/linux/bin` to our path, so to get the shell to recognize the
ack command, we can enter

```bash
source ~/.cshrc_linux
```

### Ack usage

Installation is as simple as that. Now you can quickly get useful output 
when searching large codebases. Ack is designed to only search source code
files by default, so it won't waste any time chewing through huge data 
files or binaries that happen to be in your search path. You can place
further restrictions on the filetypes searched by specifying options on
the command line.

For instance, say you are collaborating with someone and using their Matlab
scripts. You find that their scripts are working for them, but failing for
you due to tildes ( `~` ) in expressions that add folders to the Matlab 
path during execution. They expand to 
`/export/home/<THEIR_USERNAME>/rest/of/path` for them, and expand to 
`/export/home/<YOUR_USERNAME>/rest/of/path` for you. The following ack
search will quickly identify many problem lines:

```bash
cd /path/to/their/matlab/files
ack 'addpath ~'
```
or, specifying the location to search instead of changing the current
working directory:

```bash
ack 'addpath ~' /path/to/their/matlab/files
```

And you may be able to make it even faster by restricting the search to
only Matlab files:

```bash
ack --matlab 'addpath ~' /path/to/their/matlab/files
```

By default, ack will descend into subdirectories in the location you 
search. It's great for searching in large codebases to find things
you know the name of, but not the location of( e.g. calls to a 
particular function). One of my favorite command line options is
`-C` for printing additional lines of context before and after
each line where the search term was found.

```bash
ack -C 2 --cpp '->problem_function(' ~/code/large_c++_project_root_dir
```


## Installing Python

We'll download Python and build it from the source. The necessary
build tools are already available on the CN machines.

```bash
cd ~/Downloads
wget https://www.python.org/ftp/python/2.7.7/Python-2.7.7.tgz
# or
#wget --no-check-certificate https://www.python.org/ftp/python/2.7.7/Python-2.7.7.tgz
# or
#curl https://www.python.org/ftp/python/2.7.7/Python-2.7.7.tgz -o Python-2.7.7.tgz
```

The python URL may redirect to the content delivery network used by the
Python team for web hosting. This can cause wget to error out, in
which case you could use `wget --no-check-certificate <URL>` or
use `curl`, as we did above to download ack, specifying an output
file with the -o option.

```bash
tar xzvf Python-2.7.7.tgz
cd Python-2.7.7
./configure
```

Now we perform the crucial step for installing python at the user
level, rather than installing it system-wide

```bash
make altinstall prefix=~/programs/python exec-prefix=~/programs/python
```

Create a symlink in your user level bin folder to the python2.7 
executable, then refresh the shell
```bash
ln -s ~/programs/python/bin/python2.7 ~/linux/bin/python
source ~/.cshrc_linux
```

### Python usage

Feel free to fire up Python now and test it by playing around in the
interactive shell. The prompt for input is `>>> `, which changes to 
`... ` when it expects the statement may continue over additional 
lines.

```bash
python
```

You can call the exit function in Python to get out.
`>>> exit()`

If you find yourself in the old Python 2.4.3 shell, you may need

```
>>> import sys
>>> sys.exit()
```

to get out.

Set an environment variable so Python package managers install to the
correct location. Inside `~/.cshrc_linux`, add:

```bash
setenv PYTHONHOME ~/programs/python
```

As before, after saving the change to this file, log out and back in,
or run the command in the terminal.

## Installing pip, a Python package manager

**How do you install a package manager if you don't already have
the package manager, which handles this sort of thing?**

We perform a bootstrap setup of the Python package manager called
easy_install.

```bash
cd ~/Downloads
wget https://bootstrap.pypa.io/ez_setup.py
# or 
#wget --no-check-certificate https://bootstrap.pypa.io/ez_setup.py
# or
#curl https://bootstrap.pypa.io/ez_setup.py -o ez_setup.py
```

Run this script with the Python system you've installed, not the old
version that's installed for all users. If the command

```bash
which python
```

prints `/export/home/<YOUR_USERNAME>/linux/bin/python`, then you're 
ready for the next step. If it prints `/usr/bin/python`, then you may 
have missed a step previously in the guide.

```bash
python ez_setup.py
```

ez_setup also uses wget, which may fail. If necessary, you can use
`python ez_setup.py --insecure` so that it downloads without wget.

We can now use easy_install to install pip, a very popular package 
manager for Python. This is the only time we'll need to call 
`easy_install`, so instead of making a symlink for it in our bin folder,
just call it by specifying its location.

```bash
~/programs/python/bin/easy_install pip
```

Now there is a pip executable in `~/programs/python/bin`.
Create a symlink to it in our bin folder and we're done installing
Python and the pip package manager we need.

```bash
ln -s ~/programs/python/bin/pip ~/linux/bin/pip
source ~/.cshrc_linux
```

Now we're done with this section and the files we downloaded.

## Cleanup

The downloaded files can be deleted with the following commands, 
**which do not ask for confirmation, so be sure not to delete the 
wrong files.**

```bash
cd ~/Downloads

rm -f ez_setup.py
rm -f setuptools-*.zip  # Downloaded by ez_setup.py

rm -f Python-2.7.7.tgz
rm -rf Python-2.7.7     # Extracted from the .tgz
```


## Using pip to install packages

We had to do a lot of work to get Python and pip. Now that we have 
pip, installing additional packages for Python is very easy.

As an example, search pip for packages with dicom in the name

```bash
pip search dicom
```

Install the one we want, pydicom

```bash
pip install pydicom
```

## Final thought

If you install a lot of large Python packages, you may want to 
reconsider the location for the Python install. You could repeat
the steps here, altered slightly so that the install goes in 
`/data/<YOUR_USERNAME>/programs-big/python` instead.

Happy computing!
