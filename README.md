# psync

`psync` is a sync utility, the “p” stands for “patch”.

## Usage

`psync` can be useful in the following situation: you have two directories containing the same data, in two different locations. You want to be able to make some changes in one of the directories, then sync the changes to the other, without a direct connection.


### Initialisation

Before making any change to `myDirA` for instance, save the complete file list :

    $ psync init myDirA


A `.psyncstate` file is created. It contains the current list of files in `myDirA` with their modification dates.

You can now make any modification to `myDirA`.


### Creation of the patch

When you are about to leave the place A, you want to create a patch containing all the modifications made to `myDirA`. You can do it as follows:

    $ psync diff myDirA patchDir

The directory `patchDir` must exist, and must be empty. Two files are created in it: `del`, containing the list of deleted files, and `new`, a tar archive containing the modified and new files. In the meantime, the `.psyncstate` in `myDirA` is updated.


### Application of the patch

In place B, you can then apply the patch:

    $ psync patch myDirB patchDir

All the files listed in `patchDir/del` are deleted from `myDirB`, and the archive `patchDir/new` are extracted.
