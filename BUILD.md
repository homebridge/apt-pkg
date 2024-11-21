# Release workflow to Publish a refreshed homebridge-apt-pkg

## High Level Workflow to Trigger a Release

1 - Approve and merge Dependabot Pull Request ( Stage 1 and 2 )
2 - Wait about an hour for builds and testing to occur.
3 - Change Release status to Latest ( Stage 3 and 4 )

The package.json dependencies are used to manage the versions used within the release package.  And Dependabot watches the versions, and creates a pull request if a version needs updating.  Nothing else in package.json is used.  If you manually update the dependencies, package-lock.json also needs updating.

Then once a change is made to package.json in the latest branch, originated by Depandabot, the stage 1 workflow will kick off.

Release TAG is created by `release-drafter/release-drafter`, based on `.workflows/release-drafter.yml`, and not based on the package.json.

## Actions

### Stage 1 - Create a pre-release and build APT package
Average Execution time: Approx 40 minutes

This job is triggered when package.json is updated on the latest branch, and the author of the change is dependabot.

1 - Create a new release, release tag will be incremented based the latest release
2 - Build apt packages for x86_64, Arm ( RPI 32 bit), and aarch64 ( RPI 64 bit ).
3 - Apt packages are stored as an assest in the release created earlier
4 - After the build is successful, the release status is changed from `draft` to `pre-release`

### Stage 2 - Pre-Release Validation Workflow
Average Execution time: Approx 5 minutes

This job is triggered by the changing of the Release status from `draft` to `pre-release`, which is triggered by the previous job completing.

1 - This job checks that 3 apt packages are attached to the release ( x86_64, Arm ( RPI 32 bit), and aarch64 ( RPI 64 bit )).
2 - That the homebridge_*_amd64.deb apt package can be installed, and that homebridge starts.

### Stage 3 - Promote Release Package to APT Stores
Average Execution time: Approx 5 minutes

This job is triggered by the changing of the Release status from `pre-release` to `released`.  This is manual step.

1 - Release assets are downloaded from the latest release
2 - Assets are promoted to `repo.homebridge.io`
3 - Cloud flare cache is purged

### Stage 4 - Post Release Validation
Average Execution time: Approx 5 minutes

This job is triggered by the successful completion of step 3

1 - Download the current homebridge-apt-pkg for x86 and install.
2 - Check that homebridge starts