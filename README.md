# joplin-arm64-darwin-guide
Instructions to build Joplin on M1 Apple Silicon.
# This is a mirror of the post found on my website at https://noahnash.net/blog/joplin-apple-silicon

> Note: This has only been tested with a M1 Mac running Big Sur. In order to compile successfully it is *required* be running the same CPU architecture.

## 1) Install Homebrew and dependencies

### Homebrew

`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`

### Joplin Dependencies

```
brew install cocoapods
brew install python
xcode-select install
```

### Install node / npm

I successfully compiled with these versions:

Node: v16.13.1

Npm: 8.1.2

## 2) Prepare Files

### Clone Joplin repo

```
git clone https://github.com/laurent22/joplin.git 
cd joplin
```

### Modify Joplin .json files

You will need to update some dependency files to those that are M1 native. (don't worry it's easy). See: [this PR for a list of files.](https://github.com/laurent22/joplin/pull/5598)

In addition, you will have to change the target arch to arm64 in app-desktop/package.json. See: [this PR for what to change](https://github.com/laurent22/joplin/pull/5537)

### Install npm packages

```
npm install sharp
npm install Keytar —build-from-source //for some reason npm will throw errors if not done beforehand
# npm install -g @dennisameling/keytar-temp@7.4.99 // alt keytar if above does not work
# npm install sqlite3@4.1.0 —build-from-source // might not be necessary, run if build fails in step 3
```

### Set npm config flags

```
Export npm_config_arch=arm64
Export npm_target_arch=arm64
Export sdkroot=macosx
Npm config set python python3 //sqlite3 errors if not set beforehand
```

## 3) Running Joplin

### Build Joplin

```
npm install
cd packages/app-desktop
npm start 
# if it runs successfully, time to package
npm run dist --publish=never --mac --arm64
```

### Delete old install

Move the previous joplin.app to the trash. Then delete the data folder.

```
#Delete Joplin data folder: make sure you backup everything you need beforehand
rm -r ~/.config/joplin-desktop
```

### Install new dmg

A .dmg file should be in `joplin/packages/app-desktop/dist` as generated by `npm run dist` command. Extract it and run it as you would the normal x64 binary.

Open activity monitor to verify that it is using the right CPU architecture. If all goes well, you should see Joplin’s CPU type being “Apple” instead of “Intel”.

## Conclusion


I haven’t tested it extensively, but I haven’t run into any problems as of yet. But since this is not actively supported by the official project, don’t pester Joplin’s maintainers with errors in the process. If you do notice anything wrong, feel free to create an issue on this repo, and I can try and help.

> Note: due to Apple’s strict notarization and code-signing, shared prebuilt binaries will fail to boot unless you compile them yourself.

> For reference I will provide my unsigned binary on this repo for troubleshooting reasons, but don't expect it to work out of the box.

## Troubleshooting
- Make sure your NodeJS arch is arm64: `node -p "process.arch"`.
- Be sure to read through npm error logs from start to finish.
- Macs sometimes have a bug that cause it to throw enotempty errors. A *possible* fix is running: `ulimit -Sn 4096`.
- Between failed attempts: Try deleting **all** node_modules folders. `rm -rf node_modules`, and then run `npm i --package-lock-only` to update packages.
- Run `npm audit fix`,  `npm run clean`, and other commands listed in the [Joplin troubleshooting repo.](https://github.com/laurent22/joplin/blob/dev/readme/build_troubleshooting.md)

If that fails, feel free to [contact me](https:noahnash.net/contact) if you need any help, and I'll try to respond as soon as possible.


