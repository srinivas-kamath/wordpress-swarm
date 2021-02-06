This Repository contains source files for running a simple containerized wordpress setup for educational purposes.

Reader's Discretion is advised for all who wish to run these files in their development or production environments.

The files have been created while ensuring as much security and reliability as possible and the same has been tried while specifying the details in the readme files of each section.

The same standard will be maintained when adding further updates to this repository and changes will be made to existing files even as the standards evolve.

On the day of writing; This repository includes files for a simple containerized wordpress setup which does not need much configuration on the initial level and; on deleting the containers that you create via these files and the steps specified in the readme's; the data crucial for both database and code will also be deleted; hence the name of the folder "ephemeral-storage".

However; I Plan to share the steps, dockerfiles and yml files that prevent the data crucial for both database and code from deleting on the containers, swarms and stacks getting deleted; by using docker's "mounted volume" and "bind mount" features.

These updates will be rolled out soon in the future for those wanting assistance with the same.

Happy Containerizing.
