---
layout: page
title: "Continuous Integration and Delivery"
permalink: /system_design/ci_cd
---

## Notes from [Martin Fowler Continuous Integration article](https://martinfowler.com/articles/continuousIntegration.html)

Full test of version control is that you should be able to walk up with a very minimally configured environment (think new laptop of a dev) and be able to build and run the product after cloning the repository. This means you need to have int he repository the following.  Note you don't need to store binaries in the repo, but you do need to store a compiler in the repo, but we do need to know what the compiler was at a specific commit. (This may be done by storing a link to an immutable asset store)

* source code
* tests
* database schema
* test data
* configuration files
* IDE configurations
* install scripts
* third party libraries
* any tools required to build the software

By using references to an asset store, build scripts can choose to download only what is needed for a particular build.

You want to use build tools as they solve the problem of dependencies for you. For example if a step should only be run if a set of tests completes these tools can solve for you without you writing a lot of custom logic yourself. These tools can also address the issue of you rebuilding things at each build by using modifications times and a cache local or otherwise to be able to re use components that were not changed.

If everyone is pushing to mainline frequently you don't get merge conflicts that are difficult to resolve. A developer should commit to the mainline every day. Frequent commits also encourage developers to break down their work into small chunks of a few hours each. Features flags or adding code that activates functionality last can be used if you need to be adding code to the release that is part of a feature being built. Another approach for larger code changes is to build an interface for the code bing changed that can route between the old and new versions as needed.

This pushing to mainline all the time is Continuous Integration. If you have a feature branch and only merge it in when the feature is done then all other developers have to do the integration work after you add your feature, and thus its not continuous integration.

Every push to mainline should trigger a build with all the included testing.  If a build is broken it should be fixed immediately. This can be a fix forward or a revert and have someone figure out why it broke before re-adding the code.

When introducing a new feature, we should always ensure that we can rollback in case of problems. Parallel Change (aka expand-contract) breaks a change into reversible steps. For example, if we rename a database field, we first create a new field with the new name, then write to both old and new fields, then copy data from the existing old fields, then read from the new field, and only then remove the old field. We can reverse any of these steps, which would not be possible if we made such a change all at once.

Examples of CI services are

* Jenkins
* GitHub Actions
* Circle CI

One option for making slow builds more manageable is to have a **staged build pipeline**. You can do something like compile the code and run mock based tests to get a quick build and verify code is OK, and have the longer tests that require real data and databases etc. To run later. This will let you catch the quick issues fast and more subtle things may pop up later. The slower stage may not even run on every commit, it may just grab the latest and run the set of tests on that so you know what your latest good version is while having a pipeline to get subsequent versions validated. If the second build detects a bug you should mock that in your first build for the future.  

Running tests in parallel where it makes sense is another way to speed things up.

Look into Evolutionary Database Design for how to make databases work with the Continuous Integration Model