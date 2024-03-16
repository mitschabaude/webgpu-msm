# montgomery: Fast MSM in WebAssembly

_by Gregor Mitscha-Baude_

This repo contains 2 submissions:

- `submission.ts` for the main prize as originally intended: MSM over [Aleo's twisted edwards curve](https://docs.rs/ark-ed-on-bls12-377/latest/ark_ed_on_bls12_377) over the scalar field of BLS12-377.
- `submission-bls377.ts` for the side prize that was introduced midway through the competition: MSM over [BLS12-377](https://neuromancer.sk/std/bls/BLS12-377), a short Weierstrass curve.

## Using the submission

To run the following commands, first switch into the `src/submission` directory (same directory as this README). Install npm dependencies.

```
cd src/submission
npm i
```

A single command builds both submissions, such that they can be included into the main test harness:

```
npm run build-submission
```

On success, this should print:

```
built for the browser: build/web/submission.js
built for the browser: build/web/submission-bls377.js
```

These two JS files contain everything needed by the submissions (including wasm and web worker code) in a single file, bundled for the web.

In the test harness, we import these build output files instead of the TS source code:

```ts
import { compute_msm } from "../submission/build/web/submission.js";
```

We also build matching TS type definitions so `compute_msm` has proper intellisense.

### Why is there a separate build step?

Having a separate build step for the submission code enables us to keep some custom TS and build setup which webpack doesn't need to handle:

- Experimental `using` declarations and possibly other behaviour that is supported by TS only in recent releases
- Custom web worker build which inlines the worker source into the JS bundle, so that the submission behaves 'like a library' which can just resolve to a single file on import. (This will make it easier to use the submission as an npm package later on.)

### Known limitations

- Currently, there are a lot of webpack complaints about TS compile errors. They seem entirely unnecessary because the submission `d.ts` doesn't have any of these in its dependency tree. So, the solution should probably just be to make webpack somehow ignore all those TS files. I didn't have time yet to do that and didn't see it as a blocker because webpack's JS output works as expected.

- The submission currently does not work in Firefox (due to use of `Atomics.waitAsync`) and was not tested / is not expected to work in Safari or any other browser besides Chrome. I want to fix this in the near future to make the library more usable, but the prize was specifically only targeting Chrome 115.

## Direct testing of both submissions

In addition to integration with the test harness, there are various tests to run directly from the submission folder.

Two simple tests test the submission's `compute_msm()` interface directly:

- `submission-test.ts` for the twisted edwards curve
- `submission-test-bls377.ts` for the BLS curve

After having done `npm run build`, the `./run` script can be pointed at either of these to run it in Node.js:

```
> ./run submission-test-bls377.ts
ok
```

They can also be run in Chrome by calling the `./run-in-browser` script:

```
> ./run-in-browser submission-test-bls377.ts
running in the browser: build/web/submission-test-bls377.js
Server is running on: http://localhost:8000
```

This just serves a web build of the script locally, so upon navigating to http://localhost:8000 you can see the script successfully executing in the browser.

## Internal tests and scripts

There are plenty of scripts (in `/scripts`) and unit/integration tests (`.test.ts` files) to test or benchmark various aspects of this library on various curves and finite fields. All of them can be run with `./run`, and most will also work in the browser with `./run-in-browser`.

To run all `.test.ts` tests, you can also use `npm test`. (Note: at the time of writing, there are two known failing test cases unrelated to this submission.) Check out other scripts in `package.json` - they are all expected to work.
