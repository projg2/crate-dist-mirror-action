==================================
Gentoo crate-dist mirroring action
==================================

This is a GitHub action to maintain a "crate-dist" mirror. It fetches
new tags from upstream, adds a workflow based on ``crate-dist-action``
and pushes them to the repository. This workflow creates a crate tarball
using pycargoebuild_, creates a release matching the tag and uploads
the crate tarball there.

Note: due to technical limitations of GitHub, this action needs to push
to the repository via SSH rather than HTTPS. Otherwise, the added
workflow is not triggered.

.. _pycargoebuild: https://github.com/projg2/pycargoebuild/


Inputs
------

ssh-private-key
  The private key used to push to the repository. It needs to have write
  access. Required.

directories
  Space-separated list of directories to pass to pycargoebuild.
  Defaults to ``.`` (i.e. the top directory of the repository).

upstream-repo
  URL to the upstream repository. Needed only when mirroring a non-GitHub
  repository. When unset, the "parent" for the current repository will
  be used (i.e. the repository it was forked from).


Usage
-----

1. a. If source is on GitHub, fork it (include all branches, i.e. uncheck
      the "Copy the ``main`` branch only" box).

   b. Otherwise, use the "Import repository" function to create it.

2. If the action is not being hosted in ``gentoo-crate-dist`` GitHub
   organization:

   a. Get a deployment SSH key. You can either create a dedicated deployment
      key for the repository, or create a bot account with an associated key.

   b. Add the public SSH key as deployment key for the repository with write
      access, or give the bot user write access.

   c. Add an Actions secret with the private SSH key.

3. Create a new empty branch for the workflow::

       git checkout --orphan crate-dist
       git reset --hard

4. Create a GitHub workflow for the mirroring (see `Example workflow`_),
   commit it and push it.

5. Switch the default branch on the repository to your newly created branch
   (e.g. ``crate-dist``) and enable Actions.  This is necessary for scheduled
   jobs to run.

6. If you want crate dist created for an existing tag, remove it from
   the mirror, and rerun the job.


Example workflow
----------------

.. code-block:: yaml

    name: Sync with upstream
    on:
      schedule:
        - cron: "12 * * * *"
      push:

    jobs:
      sync-mirror:
        runs-on: ubuntu-latest
        steps:
          - name: Update mirror
            uses: projg2/crate-dist-mirror-action@v2
            with:
              ssh-private-key: ${{ secrets.ssh_key }}
              upstream-repo: https://gitlab.gnome.org/World/fractal.git
