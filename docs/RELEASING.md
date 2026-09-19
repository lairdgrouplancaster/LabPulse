# Releasing LabPulse

The normal release process is to publish a GitHub release, wait for GitHub's
automated publishing to finish, then run `labpulse update` on the Pi.
GitHub builds, tests, and publishes the Python package and worker container
image for you.

## Quick guide: GitHub release to updated Pi

### 1. Publish a GitHub release

Make sure the changes you want to release have been pushed to `main`. Open
[LabPulse Releases](https://github.com/lairdgrouplancaster/LabPulse/releases)
and create a new release:

1. Create a new version tag targeting `main`, such as `v0.3.9`. Choose a
   version higher than the previous release which has never been used; this
   number is only an example.
2. Give the release a title and short notes explaining what changed and any
   action users need to take.
3. Publish it as a normal release, rather than a draft or pre-release.

The tag supplies the package version automatically. Publishing the release
starts the workflow; creating or pushing a tag alone does not.

### 2. Wait for GitHub to finish

Open [Actions → Release LabPulse](https://github.com/lairdgrouplancaster/LabPulse/actions/workflows/release.yml)
and wait for all three jobs for your release to succeed:

- **Validate release artifacts** runs tests and checks the built packages.
- **Publish Python distributions to PyPI** makes the package available to pipx.
- **Publish multi-platform container** makes the matching worker image
  available to Docker on the Pi.

Wait for both publishing jobs before updating. The release page appears
before the downloads are necessarily ready. If GitHub shows a pending
environment approval, a maintainer with access needs to approve it.

### 3. Update the Pi

In the Pi's terminal, using the same account as the installation, run:

```bash
labpulse update
```

The command finds the latest version on PyPI, installs it through pipx,
regenerates the deployment, recreates the containers, waits for Home Assistant,
and runs Doctor. It preserves the live configuration, Home Assistant accounts
and history, and the current real or fake-hardware mode. Services restart
during the update.

Once it finishes, open the dashboard and check that readings are updating.
Home Assistant starts with **Test mode** enabled, so check that the notification
settings are right for normal operation.

For a non-default installation directory, use
`labpulse --live-dir /path/to/installation update`. To select a particular
published version, use `labpulse update X.Y.Z`, without the tag's leading `v`.
See the [operator update guide](USER_GUIDE.md#updating-labpulse) for backup and
recovery details.

## If something fails

**A GitHub job fails:** open that job's log. If the cause is permissions or a
temporary publishing failure, fix it and rerun the failed job. One publishing
job can succeed while the other fails, so check both before updating the Pi.
If the released code needs changing, publish a new version; do not move or
reuse the published tag.

**The Pi update fails:** read the error to see which stage failed and follow
[update troubleshooting](TROUBLESHOOTING.md#update-failed-or-sms-worker-is-offline).
An update can partly complete; it does not automatically roll back.

The implementation lives in the [GitHub release workflow](../.github/workflows/release.yml)
and [update command](../src/labpulse/control.py). Local builds and development
images are covered in [Development](DEVELOPMENT.md#host-code-and-runtime-images).
