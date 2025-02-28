---
parent: VAYS
nav_order: 4
---

# Development

## Branches

There are three core branches, `development`, `testing` and `main` (release). 

## Tagging
To release a new version, it needs to be tagged. There are currently two places, in which the version number needs to be updated.

One is git itself. There, tagging is used to increase the version.
The other one is in package.json, where npm and the rest of the source code (including the build process) gets the version from there.

To maintain both without worrying about both, check out the next paragraph.



## Releasing a test version
A test version is released by 

- merging from development into testing
- tagging this commit
- push the branches and tag, using a release candidate (vX.YrcZZ)

### Doing this automatically
To do the above automatically, do the following:
First, go into the development branch. Then

```sh
./scripts/incr-version.sh rc --merge
```

Try it! This will first preview what changes it will do. Without your confirmation, it does not change anything. Then, it will automatically merge, tag and push.

{: .warning}
This will make another commit, committing anything that is not yet committed in this branch! Make sure you stash your changes if necessary! It will change the `package.json` file (increasing the version there. Also, for convenience only, it will also edit `/rsc/version.tsx`. Note that this is for convenience only, making sure that the build process does not make new changes to this file in the dev environment. (The build process writes the version in `package.json` into the `/rsc/version.tsx` file such that it can be included in the codebase and displayed to the user.)


If you want to do merging of `development` into `testing` yourself, omit the `--merge` option.

{: .warning}
After release, the tool leaves you in the `testing` branch.

## Releasing a minor version
Doing a minor release (v0.X, v1.X, with X upgrading.) is the same process above, if done manually.

The script you can use is 
```sh
./scripts/incr-version.sh minor --merge
```

See the notes above.


### Upgrade Environment

Use 

```sh
npm update --save
npm update --save-dev
```
Don't join the two. 
