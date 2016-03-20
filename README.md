# psync

`psync` is a sync utility, the “p” stands for “patch”. It is designed to keep synchronised two or more directories, distant from each other, without a direct connection. Another directory called *patching directory* is used to synchronise them.

## How it works

Each directory contains a *state file* which is basically a gzipped text file containing:

* A 160-bit hex-encoded unique identifier
* A timestamp
* The list off all files with their modification timestamp

After having made some modifications to one of the directory, it is possible to check the differences with the saved state file and make a patch to apply them to the other directories in sync. A patch contains a list of deleted files, and an archive of new or modified files.

## Usage

### Initialisation

There is several ways to initialise the synchronisation system. The first is to create an initial state file in one directory:

    $ psync init "Directory A"

Then, use a traditional sync utility like `rsync` to synchronise the directory with other directories, in order to make them be in the same initial state. **As `psync` uses `ctime` to check the modifications, when the directory is copied *via* another utility than `psync`, the state file must be then updated as follows:**

    $ psync update "Directory B"

Another solution is to create an initial patch:

    $ psync diff "Directory A" PatchDir

As there is no state file in `Directory A`, an empty one is created with timestamp 0, and the patch for timestamp 0 contains all the files in the directory. It can be later applyed to patch an empty directory:

    $ psync patch "Empty Directory B" PatchDir -f <uid>

The `<uid>` value is displayed by the first diff.

### Creation of the patch

After the synchronisation system has been initialised, it is possible to make changes to one of the directories in sync (but **only** one of them).

Say we are in place A and we have made some changes to `Directory A`. Before leaving the place A, we may check the differences with the saved state and make a patch:

    $ psync diff "Directory A" PatchDir

If it does not exist yet, a directory named from the UID of the synchronisation system is created. In this directory, a subdirectory is created for the patch. It is named from the timestamp of the previous saved state. The patch contains three files:

* `del`, which contains the list of deleted files;
* `new`, a tape archive containing the new and modified files;
* `newdir`, a tape archive containing the new and modified directories.

### Application of the patch

Once in place B, we may now apply all available patched to `Directory B`:

    $ psync patch "Directory B" PatchDir

`psync` looks for a state file in `Directory B` to get the UID and initial timestamp. Then, it looks for a corresponding patch in `PatchDir`. If there is one, the following actions are performed:

* all the files listed in `PatchDir/<uid>/<timestamp>/del` are deleted from `Directory B`;
* the archive `PatchDir/<uid>/<timestamp>/new` is extracted in `Directory B`;
* the archive `PatchDir/<uid>/<timestamp>/newdir` is extracted in `Directory B`.

As the directories are extracted after files, their modification time is preserved.

The user is then prompted the possibility to delete the patch. This is the default behaviour, but it can be useful to save the patch if there is more than two directories to synchronise.

Finally, `psync` checks if there is another patch related to the new timestamp and apply it if appropriate.
