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

**Known limitations**. Currently, there are a lot of webpack complaints about TS compile errors. They seem entirely unnecessary because the submission `d.ts` doesn't have any of these in its dependency tree. So, the solution should probably just be to make webpack somehow ignore all those TS files. I didn't have time yet to do that and didn't see it as a blocker because webpack's JS output works as expected.
