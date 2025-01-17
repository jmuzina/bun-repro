Two bin scripts are defined in `bin-script-definitions`' `package.json`:
- bin-script-test-ts
- bin-script-test-js

The typescript bin script runs a script that is generated by `tsc` and thus is not available at the time of dependency installation.
The javascript bin script runs a script that is already present before dependency installation.

On some machines (currently I've only noticed this occurring on my M2 MBP), this causes the 
typescript bin script to not be added to the `node_modules/.bin` directory, until `bun run build` is called.

In this setup, `bun i` runs `bun run build` via the `prepare` script, so the .bin script will be created by running `bun i` twice.

This can be observed by running `bun i` from a clean monorepo root and then viewing the node_modules/.bin directory.

First time running `bun i`:
```
➜  bun-repro git:(main) ✗ bun i
➜  bun-repro git:(main) ✗ ls node_modules/.bin | grep bin-script-
bin-script-test-js
```
Second time running `bun i`:
```
➜  bun-repro git:(main) ✗ bun i
➜  bun-repro git:(main) ✗ ls node_modules/.bin | grep bin-script-
bin-script-test-js
bin-script-test-ts
```

This appears to also happen when using `npm` and `yarn` so I suspect it is not an issue with any package manager,
but rather a problem with this specific approach of using a bin script that is generated by a build step on the prepare hook.

I'm not sure why this only occurs on my MBP and not my System76 laptop that runs PopOS/Ubuntu; 
possibly one of these machines is faster than the other and there is a race condition here. 

A proper solution to this should be investigated, but for now it is sufficient to run `bun i` twice, especially in a context where
`nx` caches the build step so we only build once, even though we run `bun i` twice.