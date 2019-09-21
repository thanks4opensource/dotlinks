dotlinks: simple management of configuration files across multiple computers
=============================================================

*dotlinks* is a very simple, lightweight technique for managing configuration files, typically but not limited to "dot" files in users' home directories. It allows a single, unmodified $HOME directory tree to be shared, cloned, archived, backed up, version controlled, etc. across multiple computers and operating systems, without requiring reconfiguration after copying to a new computer or adding new configuration files or directories.



<a name="contents"></a>
Contents
--------
* [License](#license)
* [Problem definition](#problem_definition)
    * [Background](#background)
    * [Multiple computers, different dotfiles](#multiple_computers_different_dotfiles)
* [The *dotlinks* solution](#the_*dotlinks*_solution)
    * [The technique](#the_technique)
    * [Notes and requirements](#notes_and_requirements)
    * [Usage](#usage)
    * [Limitations and testing](#limitations_and_testing)
    * [Example implementation](#example_implementation)
* [Alternative solutions](#alternative_solutions)




<a name="license"></a>
License
-------
*dotlinks*: simple management of configuration files across multiple computers

Copyright (c) 2019 Mark R. Rubin

*dotlinks* and the technique(s) it describes are licensed under the terms of the Creative Commons [Attribution-ShareAlike 4.0 International](LICENSE_cc-by-sa.txt), also available at <https://creativecommons.org/licenses/by-sa/4.0/>.



<a name="problem_definition"></a>
Problem definition
------------------

<a name="background"></a>
#### Background
Applications running on UNIX and UNIX-derived operating systems (Linux, BSD) frequently use configuration files to control their behavior. These files are usually stored in the user's home directory and have names beginning with "." to prevent them from appearing in normal directory listings. The so-called "dotfiles" typically have the same name as the application that uses them, and although there are variations on the scheme (a "dot directory" containing multiple configuration files, or a directory with the un-dotted application name in the `$HOME/.config` sub-directory), all basically perform the same function.

<a name="multiple_computers_different_dotfiles"></a>
#### Multiple computers, different dotfiles

If a user uses an application on multiple computers, they may need different versions of its dotfile(s) due to differences in the hardware and/or software environments, or versions of the application, on those computers. At the same time users frequently want to have the same data, files, directory structure, etc. in their `$HOME` directory across the different computers.

This presents a data management problem, particularly in regards to backups/archives of the `$HOME` data. Either multiple backups must be maintained, one for each computer, or methods put in place for re-configuring the dotfiles on each computer when copied from another or restored from backups. (See also [Alternative solutions](#alternative_solutions), below).




<a name="the_*dotlinks*_solution"></a>
The *dotlinks* solution 
-----------------------

#### The technique <a name="the_technique"></a>

*dotlinks* attempts to solve the problem by use of multilevel symbolic file links (symlinks) created according to the following scheme:

1. All dotfiles which need to differ between computers are replaced by symlinks.
2. Each symlink points to a directory path identical to the original one in `$HOME` but rooted in a user-specific directory outside `$HOME`. For example:
    * A `/home/username/.emacs` file is replaced by a symlink to  `/home/dotlinks/username/.emacs`
    * A `/home/username/.config/some_application/` directory by a symlink to `/home/dotlinks/username/some_application/`
    * A `/home/username/.config/some_application/themes/my_light_theme.dat` file by a symlink to `/home/dotlinks/username/.config/some_application/themes/my_light_theme.dat`.
3. Each `/home/dotlinks/username` entry is a symlink to a `/home/username/dotlinks/a_computername/` directory, where *a_computername* is different on each computer. Each user's `$HOME/dotlinks/` directory contains multiple "a_computername" directories, one for each computer.
4. Each of the `/home/username/dotlinks/a_computername/` directories contains files and sub-directory trees matching the original ones in $HOME. For the symlinks in #2, above, these would be:
    * `/home/username/dotlinks/a_computername/.emacs`
    * `/home/username/dotlinks/a_computername/.config/some_application/`
    * `/home/username/dotlinks/a_computername/.config/some_application/themes/my_light_theme.dat`.
5. Optionally, if many computers need the same version of a dotfile and only a few computers are exceptions, a directory such as `$HOME/dotlinks/generic/` can be created and populated with the basic versions of the dotfiles and directories. Then, for example, the majority of the `/home/username/dotlinks/a_computername/.some_app_config` files and directories can be direct symlinks to `$HOME/dotlinks/generic/.some_app_config`. This avoids unnecessary duplication of files.


<a name="notes_and_requirements"></a>
#### Notes and requirements 

* The technique inherently supports multiple users.

* All filenames and directory paths in the above description are arbitrary. The names of the symlinks in `/home/dotlinks/` do not need to match their targets in the user `$HOME/dotlinks/` directories although there is little reason to do otherwise.

* The directory `/home/dotlinks/` as described in this documentation could be anywhere and named anything, however it **must** be outside of any user's `$HOME` directory tree. It would typically be created/configured via administrator access (root or `sudo`) but could be in any parent directory writable by the user doing the installation, although care would be required in the latter case that the directory was permanent. Another mandatory requirement is that the directory path be the same on all computers where dotlinks is being used.

* The technique obviously requires an operating system and filesystem that supports symbolic links or other file- and directory-linking methods. Note that the linking must be name based --- direct low-level filesystem links (hard links on UNIX-like systems) will not work for the dotlinks method.  

* Only configuration files that need to differ among computers need/should be implemented with the technique. There is no harm in adding others but doing so creates unnecessary complexity due to duplication of files or proliferation of symlinks.

<a name="o_notation"></a>
* Although the problem being addressed is inherently *O(n<sup>2</sup>)* --- there need to be *N\*M* configuration files for *N* applications on *M* computers --- the dotlinks solution only adds linear *O(m)* additional complexity for *m* different computers. Each of the *N* applications' dotfiles/dirs link via a single `/home/dotlinks/username/` path.

* Unnecessary dotfiles, *computername* directories, users, etc. do not need to be implemented, which somewhat reduces the number of *O(n<sup>2</sup>)* files required. If a user is not using *the_application* on *some_computer*, the `$HOME/dotlinks/some_computer/dotfile_for_the_application needn't exist. (This will result in a dangling symlink, but one that will never be referenced.)

* The technique is version control and backup system agnostic. For example, `RCS` directories and/or `,v` files, and `.git` repository directories, can be arbitrarily included. In the RCS case a decision needs to be made whether to symlink to a common `,v` RCS file and use RCS branches to differentiate the various computers' versions, or use separate `,v` files, each diverging from an ancestor as described in ["Usage"](#usage), below. If using GIT, each computer would use a separate clone of the master GIT repository.


<a name="usage"></a>
#### Usage 

<a name="to_initialize_dotlinks"></a>
To initialize dotlinks, first:

1. Create the `/home/dotlinks/` directory on every computer.
2. On each computer create `/home/dotlinks/username` symlinks pointing to each user's `$HOME/dotlinks/computername/` directory.
3. Create the  `$HOME/dotlinks/computername/` directories, one for every *computername* in the system, in each user's `$HOME/dotlinks/` directory.

When a user decides that a dotfile or directory needs to be different across multiple computers, they should:

1. Copy the dotfile (or recursively copy the configuration directory) to either:
    1. Each of the `$HOME/dotfiles/computername`/ directories, or ...
    2. To a `$HOME/dotfiles/generic/` directory, and create symlinks in each of the `$HOME/dotfiles/computername/` directories directly to it.
2. Edit the `$HOME/dotfiles/computername/.application_dotfile` as required, first breaking the symlink and making a full copy if using `$HOME/dotfiles/generic/` as per #2 above.
3. Propagate the additions/changes to `$HOME/dotfiles/*` to each computer and to the `$HOME` backups and archives, as required.

To add a new computer, perform the steps listed above (in [To initialize dotlinks](#to_initialize_dotlinks)) using a *new_computername*. Each user can then:

1. Recursively copy the most similar pre-existing `$HOME/dotfiles/other_computername/` directory to a new `$HOME/dotfiles/new_computername/` directory.
2. Break or create symlinks to `$HOME/dotlinks/generic/` in `$HOME/dotfiles/new_computername/` as necessary.
3. Edit the non-symlinked dotfiles and directories to configure for *new_computername*.
4. Propagate the new `$HOME/dotfiles/new_computername/` directory to their $HOME directories on the  other computers and their `$HOME` backups and archives.




<a name="limitations_and_testing"></a>
#### Limitations and testing

As of the current date (September 2019) the author has used the dotlinks technique continuously for over three years without any significant problems.

A major limitation, however, is with applications that both write as well as read their dotfile(s), and do so by deleting and recreating them instead of opening them for writing or read+write. Such applications  will destroy the dotlinks symlinked dotfile in the process. An example of an application that does this is the `xscreensaver` utility on Linux, when it is re-configured using its own built-in GUI.

Additionally, some applications may explicitly detect whether their configuration files and/or directories are symlinks and act on that knowledge. The author has seen `ssh` do this (perhaps as an additional security measure) which is particularly unfortunate as the `ssh` credentials directory is a perfect use-case for a technique like dotfiles because SSH credentials differ between each computer.



<a name="example_implementation"></a>
#### Example implementation:

On computer named "workstation" running openSUSE Leap 15.1:

        $ pwd
        /home/dotlinks
        $ file *
        thnks4oss: symbolic link to /home/thnks4oss/dotlinks/openSUSE-Leap-15.1-workstation
        
        $ pwd
        /home/thnks4oss
        $ file .emacs .fvwm .zshrc bin/term
        .emacs:   symbolic link to /home/dotlinks/thnks4oss/.emacs
        .fvwm:    symbolic link to /home/dotlinks/thnks4oss/.fvwm
        .zshrc:   symbolic link to /home/dotlinks/thnks4oss/.zshrc
        bin/term: symbolic link to /home/dotlinks/thnks4oss/bin/term

On computer running Ubuntu 18.04 (not running FVWM nor needing custom `term` program):

        $ pwd
        /home/dotlinks
        $ file *
        thnks4oss: symbolic link to /home/thnks4oss/dotlinks/ubuntu-18.04
        
        $ pwd
        /home/thnks4oss
        $ file .emacs .fvwm .zshrc bin/term
        .emacs:   symbolic link to /home/dotlinks/thnks4oss/.emacs
        .zshrc:   symbolic link to /home/dotlinks/thnks4oss/.zshrc

On all computers: 

        $ pwd
        /home/thnks4oss/dotlinks
        $ tree -a --charset ascii --noreport
        |-- generic
        |   |-- .emacs
        |   |-- .zshrc
        |   |-- RCS
        |   |   |-- .emacs,v
        |   |   `-- .zshrc,v
        |   |-- bin
        |   |   |-- term
        |   |   |-- RCS
        |   |       `-- term,v
        |-- openSUSE-Leap-15.1-laptop
        |   |-- .emacs
        |   |-- .fvwm
        |   |   |-- RCS
        |   |   |   |-- config,v
        |   |   |   `-- xdg_menu.rc,v
        |   |   |-- config
        |   |   `-- xdg_menu.rc
        |   |-- .zshrc
        |   |-- RCS
        |   |   |-- .emacs,v
        |   |   `-- .zshrc,v
        |   |-- bin
        |   |   |-- term
        |   |   |-- RCS
        |   |       `-- term,v
        |-- openSUSE-Leap-15.1-workstation
        |   |-- .emacs
        |   |-- .fvwm
        |   |   |-- RCS
        |   |   |   |-- config,v
        |   |   |   `-- xdg_menu.rc,v
        |   |   |-- config
        |   |   `-- xdg_menu.rc
        |   |-- .zshrc
        |   |-- RCS
        |   |   |-- .emacs,v
        |   |   `-- .zshrc,v
        |   |-- bin
        |   |   |-- term
        |   |   |-- RCS
        |   |       `-- term,v
        |-- openSUSE-Leap-42.1
        |   |-- .emacs -> /home/thnks4oss/dotlinks/generic/.emacs
        |   |-- .fvwm
        |   |   |-- RCS
        |   |   |   `-- config,v
        |   |   `-- config
        |   |-- .zshrc
        |   |-- RCS
        |   |   `-- .zshrc,v
        |   |-- bin
        |   |   |-- term
        |   |   |-- RCS
        |   |       `-- .term,v
        |-- ubuntu-18.04
        |   |-- .emacs -> /home/thnks4oss/dotlinks/generic/.emacs
        |   |-- .zshrc -> /home/thnks4oss/dotlinks/generic/.zshrc
        |   |-- bin
        |   |   `-- term -> /home/thnks4oss/dotlinks/generic/bin/term




<a name="alternative_solutions"></a>
Alternative solutions
---------------------

The author conceived and implemented the dotlinks technique(s) independently, and prior to the current date (September 2019) was not aware of any similar work.

After long use (see [Limitations and testing](#limitations_and_testing), above) and deciding that the technique might be useful to others, research turned up several alternative published solutions. These include:

* <https://stackoverflow.com/questions/1413049/managing-user-configuration-files-across-multiple-computers>
* <https://github.com/jeviolle/slack>
* <https://github.com/RichiH/vcsh>
* <https://github.com/GMadorell/dotfiles_deprecated>
* <https://github.com/MarcelloNicoletti/dotfiles>

There are also large-scale commercial and open-source systems such as:
* <https://cfengine.com/>
* <https://puppet.com/>
* <https://www.ansible.com/>

A GNU FOSS project:
* <https://www.gnu.org/software/stow/>

The Linux `/etc/alternatives` directory and `update-alternatives` command.

Most of the above use a version control repository such as GitHub to install and update dotfiles on the different computers, potentially with different versions. Others consist of programs to create symlinks and do not address the problem of version differences. The large-scale systems are aimed at installation and version control of system-level software packages and their configuration.

The `update-alternatives` command and its use of two-level linking from `/usr/bin` to `etc/alternatives` and back (or to other directories) is very similar to the dotlinks approach, although aimed at system binaries instead of user dotfiles. It also does not include the concept of multiple simultaneous configurations with an active one chosen on a per-computer basis as in dotlinks' `/home/dotlinks/username` symlinks.

GNU *Stow* also has much in common with dotlinks. Its concept of "tree folding" is the same as dotlinks' *O(m)* (see
[above](#o_notation)) single links in `$HOME/dotlinks` (instead of one for each dotfile or directory). However it does not seem to use or address dotlinks' multiple computer technique, and additionally seems largely aimed at providing tools to automatically create and manage the links. By contrast, the intent of dotlinks is that the links are largely static and unchanging, and that managing them can easily be done manually as per [Usage](#usage), above. Nevertheless, despite the claim that dotlinks can be used for other purposes than `$HOME/.dotfile` management, the author would likely use *Stow* for package management if needed.

In any case, improvements to dotlinks and suggestions for this document are welcome.
