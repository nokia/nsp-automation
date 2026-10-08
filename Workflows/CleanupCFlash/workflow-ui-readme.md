# CleanUp Compact Flash

## Name

cleanupCFlash

## Description

The cleanupCFlash workflow can be used to remove old files from the NE filesystem. Any files in the target directory that are older than *deleteAge* in seconds will be deleted. In addition, the workflow also supports the standard *dryRun* input attribute to allow the user to test which files will be deleted.

## Usage

The cleanupCFlash workflow takes the following as input parameters:

* neName — The name of the device
* dir — The target directory to be cleaned, default *cf3:*
* deleteAge — The minimum age of the file in seconds before deletion, default *3600*

In addition, the workflow also supports the standard *dryRun* input attribute.
