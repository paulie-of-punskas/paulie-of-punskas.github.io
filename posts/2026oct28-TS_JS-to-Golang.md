# GHA action - TS/JS to Golang

You'll probably ask *why*. Well, *why not*?

In January this year [1], I released a GitHub action, for fetching programming language versions, that were approaching End Of Life support. All versions up to v1.1.0 were written using Typescript. Anyone who has done something with Javascript or Typescript, had to deal with npm, transpiling/packaging nuances and possible event of a supply chain attack. As I knew Golang before, I decided to replicate the code and build a binary. Continue reading, to follow my rationale and kind of a journey.

# The < v1.1.0

So, I prior to v1.1.0 [2], I had created couple of guardrails for npm environment. One can use static code analysis tools, monitor CVE registry, setup Dependabot, use LLMs and much more. In my case, I had setup a project on Snyk for code analysis and monitored updates for dependencies - Dependabot and with `npm audit`. I was satisfied - I was using mature packages, directly interfaced ones were developed by GitHub folks [3]. All of them are actively developed and well known. package.json, contained only a minimal number of dependencies, that were crucial for development and functionality [4]:

```JSON
  "dependencies": {
    "@actions/cache": "6.3.0",
    "@actions/core": "^3.0.1",
    "@actions/github": "^9.1.1"
  },
  "devDependencies": {
    "@eslint/compat": "^2.0.2",
    "@jest/globals": "^30.2.0",
    "@rollup/plugin-commonjs": "^29.0.0",
    "@rollup/plugin-json": "^6.1.0",
    "@rollup/plugin-node-resolve": "^16.0.3",
    "@rollup/plugin-typescript": "^12.3.0",
    "@types/jest": "^30.0.0",
    "@types/node": "^26.2.0",
    "jest": "^30.2.0",
    "prettier": "^3.8.1",
    "rollup": "^4.57.1",
    "ts-jest": "^29.4.6",
    "ts-jest-resolver": "^2.0.1",
    "tslib": "^2.8.1",
    "typescript": "^5.9.3"
  }
```

Quite simple, as you can see.

# "The Good, The Ugly and The Gopher"

## The Good

Then, I was hit by the PRs opened by it, that were not really related to the code I wrote. Sure, they were transitive dependencies (i.e. fast-xml-parser, undici) that I was not aware of, but I had review the updates to be aware of how potential security exploit might influence the code. Thus, I either reviewed and closed or reviewed and approved the PRs that Dependabot created.

## The Ugly

Quite soon, I got gently hit with npm reality:

- some of updated dependencies started to break the package. I prevented it by extending the CI pipeline, so it's run on any change to package.json and/or package-lock.json.
- unexpected breaking changes i.e. use of TypeScript 7 with ts-jest (as of writing of this text)
- constant monitoring of dependencies is interesting, but more tiring than expected. Each week, expect at least 1 PR with an update.
- hats off to unpaid maintainers for working on the Open-Source software. They do not get enough recognition.

I had an idea, that if a runner allows a bash command to be executed, it *should* also allow me to run a precompiled binary. In this case, I could actually use a programming language, without the npm' wild west. I considered other programming languages I knew, but there was always a "but":

- Java - why not:
    - but do I really need a JVM for this simple app
    - difficult to say, what size the binary would be
    - 100% OOP boilerpalte, while majority of code is sufficient can be implemented with functional paradigm and elements of OOP
- Python - PyPI reality oh boy. I would stick to npm ;)
- Shell scripting - it should be possible, but it's easier to do many things using appropriate tools. Shell scripting is too difficult to maintain in this case and lacks testing framework.
- R or SAS - no statistical computation here
- VBA - (skull icon)
- C - would be cool, fast, but is it worth it?
- Golang - *why not?*

# The Gopher

Golang is mature and offers a significant functionality out of the box. It's fast, developed with a simplicity at it's core [5], that allows anyone to build a CLI tool. Testing framework is there, mocking is easier that with `jest`. What could go wrong?  
.  
.  
.  
Nothing. Just a flexibility is required.

# Lessons learned

- as of writing this text, by default GitHub Actions runners are running on Ubuntu. Make sure, to build a binary for Linux (not macOS or Windows)
- integration tests remotely - at first, for testing purposes, I was compiling and committing binary along with the the code changes. That actually can be simplified, by building it remotely and the running integration tests.
- build it remotely, commit it remotely - I had to update CD job, so that github-actions bot could commit the built binary to the same release PR
- integration with GitHub Actions - while running binary is straightforward, integrating it with the runner and GitHub Actions is not. @actions/toolkit is not available for Golang and this required me to rewrite the action.yaml to a composite on
- for cache creation and restoring I had to rely on https://github.com/actions/cache
- setting up CLI flags can be tricky and might lead to reworks. But, you learn something
- will runners allow binary execution - that's an open question for now

# Comparison

- similar size ( ~8.5MB) for both transpiled JS (+ map) and compiled Golang binary
    - for JS I had used rollup as it's mature and offers treeshaking. I also tested ncc and esbuild, but I got tired with setting up TS all over again.
- execution and build time is a bit longer for Golang, however several seconds are negligible in this case
- I had to embed an external file into a binary. Otherwise it would be unreachable and code would panic
- no Dependabot alerts, no exploits found by Snyk, GraphQL (additionally setup) is quiet as well

# Conclusion

Integration of GitHub Actions with Node is superb. A significant amount of boilerplate can be overcome, thanks to a family @actions/toolkit. This allows to enclose *everything* in one ecosystem. However, it's also possible to diverge slightly from it and keep the functionality. With a bit of flexibility, it can be be implemented with vanilla Golang and appropriate GitHub action as well. If cache was not required to be used, one could rely solely on Golang.

Given the peace of mind I got, I am sticking with Golang. Thank you for reading it. No LLM was used for producing this text.

**Sources:**  
[1] https://github.com/paulie-of-punskas/get-lts-versions/tree/v0.1.0  
[2] https://github.com/paulie-of-punskas/get-lts-versions/tree/v1.1.0  
[3] https://github.com/actions/toolkit  
[4] https://github.com/paulie-of-punskas/get-lts-versions/blob/v1.0.6/package.json  
[5] https://dave.cheney.net/2020/02/23/the-zen-of-go