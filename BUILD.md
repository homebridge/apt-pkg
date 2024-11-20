# Release workflow to Publish a refreshed homebridge-apt-pkg

## High Level Workflow

1 - Approve and merge Dependabot Pull Request
2 - Wait about an hour for builds and testing to occur.
3 - Change Release status to Latest


The package.json dependencies are used to manage the versions used within the release package.  And Dependabot watches the versions, and creates a pull request if a version needs updating.

Then once a change is made to package.json, originated by Depandabot, the github action release workflows will kick off.

## Actions

### After update of package.json by dependabot - create a release and build APT package

This job is triggered when package.json is updated on the latest branch, and the author of the change is dependabot.

1 - Create a new release, release tag will be incremented based the latest release
2 - Re-Use the workflow `_build_package_and_store.yml` to build apt packages for x86_64, Arm ( RPI 32 bit), and aarch64 ( RPI 64 bit ).
3 - Apt packages are stored as an assest in the release created earlier
4 - After the build is successful, the release status is changed from `draft` to `pre-release`

### Pre-Release Validation

This job is triggered by the changing of the Release status from `draft` to `pre-release`, which is triggered by the previous job completing.

1 - This job checks that 3 apt packages are attached to the release
2 - That the homebridge_v1.3.10_amd64.deb apt package can be installed, and that homebridge starts

### Promote Release package to APT Stores

This job is triggered by the changing of the Release status from `pre-release` to `released`.  This is manual step.

1 - Release assets are downloaded from the latest release
2 - Assets are promoted to `repo.homebridge.io`
3 - Cloud flare cache is purged

### Validate apt pkg release post build and publish

This job is triggered by the promotion of new code to the APT Store.

1 - Download the current homebridge-apt-pkg and install.
2 - Check that homebridge starts