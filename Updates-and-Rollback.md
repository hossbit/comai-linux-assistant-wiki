# Updates and rollback

[Documentation home](Home.md)

## Preview before applying

```bash
comai update --check
comai update
comai update --ref v2.10.0
```

Updates select the newest stable release tag and resolve it to an immutable
commit before fetching or executing the installer. A full 40-character commit
can also be selected with `--ref`. Branch names such as `main` are not accepted
by this update command.

Git checkouts must be clean and the target must be a fast-forward. Commit or
stash local changes first. Installed copies preserve configuration and retain a
private adjacent application snapshot.

## Return to the previous application

```bash
comai update --rollback
```

For installed copies, rollback restores the exact application snapshot while
keeping current configuration. For Git checkouts, it switches to the recorded
commit in detached mode. External model directories are unaffected. Backups are
retained for inspection and manual cleanup.

## Without Git

Archive updates require an explicit full commit plus a trusted
`COMAI_TARBALL_SHA256`; missing or mismatched checksums stop installation. Unsafe
archive paths, links and special files are rejected.

ComAI releases currently use GitHub's standard source downloads; there is no
additional custom source archive, `provenance.json` or `SHA256SUMS` asset in
v2.10.0. Installing Git is the straightforward way to use verified commit updates.
Do not invent a checksum or bypass verification.
