# Contributing

This is a personal fork of
[pop-os/cosmic-settings](https://github.com/pop-os/cosmic-settings). Upstream's
contribution rules do not apply here.

## AI-assisted contributions

Pull requests containing AI-assisted work — including LLM-generated code,
comments and descriptions — are welcome in this fork.

When opening a pull request:

- say which parts were AI-assisted (one line in the description is enough);
- understand the change and be able to answer review comments;
- keep the change focused, and run `just check` and the relevant tests;
- disclose anything you are unsure about; it is easier to review than a surprise.

## License

By submitting a contribution you agree that it is licensed under the GNU Public
License 3.0, as described in the [README](./README.md#license).

## Syncing with upstream

`origin` is `pop-os/cosmic-settings`; `fork` is this repository. `master` is kept
clean and mirrors upstream, and feature work lives on its own branch.

```sh
# update master
git fetch origin
git checkout master
git merge --ff-only origin/master
git push fork master

# move the feature branch on top of the new master
git checkout feat/sound-output-port
git rebase master
git push --force-with-lease fork feat/sound-output-port
```

Conflict hot spots: the "This fork" section in `README.md`, the pull request
template, and the sound page (`pages/sound/src/model.rs`,
`cosmic-settings/src/pages/sound/mod.rs`). Keep the output-port logic and fold
the upstream changes in. If upstream changes the `ActiveRoute` semantics or the
`set_route()` signature, the build fails and the call sites need updating.

## Installing this build locally

Do **not** remove the packaged `cosmic-settings`: `cosmic-session` depends on it.
Install this build to `/usr/local` instead. `/usr/local/bin` comes first in
`PATH`, so the desktop entry (`Exec=cosmic-settings`) starts this binary, and
package upgrades do not touch it.

The binary is installed under this fork's name, with a `cosmic-settings`
symlink so existing desktop entries and panel buttons keep working:

```sh
cargo build --release -p cosmic-settings
sudo install -Dm0755 target/release/cosmic-settings /usr/local/bin/cosmic-allow-ai-settings
sudo ln -sfn cosmic-allow-ai-settings /usr/local/bin/cosmic-settings
```

The packaged app stays installed as a fallback. Re-run these commands after
pulling upstream changes.

`just install` (with `prefix=/usr/local`) additionally installs the desktop
file, metainfo, icons and the default schema, which is only needed if the
packaged files are missing.
