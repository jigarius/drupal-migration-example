# Installation data

This directory contains exports for certain configuration parameters which are imported during the installation of the module.

* The `migrate_plus.*` files are for setting up migrations for the module.
* The other files are to ensure that when the module is installed, all relevant node types, field types, etc installed with it.
  * If you do not have a _standard_ install, you might have to remove the `.ignore` suffix from some of the `*.yml.ignore` files in this directory.
