---
 title: Public CI
 cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1674831252879/5Igyt1j_j.jpeg?auto=compress
 tags: database, ci, github-actions
 domain: blog.venicedb.org
 publishAs: felixgv 
---

For the past few years, the Venice team at LinkedIn has been maintaining a Jenkins cluster to run builds. This has worked well and it helps improve the team’s productivity, but now that the project is open source, it is preferable to have a publicly accessible CI solution. For this, we have chosen GitHub Actions. This is a quick overview of what we have done so far in this space, and what we're hoping to do later.

[Sushant](https://www.linkedin.com/in/sushantmane/) kicked us off by [configuring the actions](https://github.com/linkedin/venice/pull/115), which was merged at the end of 2022. However, because the GitHub Actions VMs are fairly weak, the build took over 4 hours of machine time, and even with basic parallelism, it still took over 2 hours of wall clock time. This was a degradation compared to the internal CI which could run the whole suite on a single machine in about 1 hour, so the first order of business was to speed up the build.

Historically, we had not been very diligent about what tests went where, so we had integration tests (i.e. anything that uses our [ServiceFactory](https://github.com/linkedin/venice/blob/main/internal/venice-test-common/src/integrationtest/java/com/linkedin/venice/integration/utils/ServiceFactory.java)) littered everywhere. We [consolidated all of them into a single module](https://github.com/linkedin/venice/pull/155), which means now every other module is strictly reserved for unit tests. The unit tests take about half an hour to run, so we put those in a single task, and went about splitting the now consolidated integration tests into a bunch of smaller buckets so that they could run in parallel. It is hard to bin pack optimally (and we can still do better in that regard) so the goal was not necessarily to make all buckets take the same amount of time, but merely that no bucket took longer than the unit tests.

In the end, due to limitations in [GitHub's implementation of YAML](https://github.com/actions/runner/issues/1182), we ended up [dynamically generating the GitHub Actions](https://github.com/linkedin/venice/pull/161) as part of our Gradle build. There is undoubtedly a better and cleaner way to do this, but for the time being, this works, and it lowers the CI's wall clock time to about half an hour; faster than the internal CI!

[Adam](https://www.linkedin.com/in/xinchen8/) then added static analysis so that violations of [Spotbugs rules](https://github.com/linkedin/venice/pull/170) as well as degradations in [code coverage](https://venicedb.org/docs/dev_guide/code_coverage_guide) cause the build to fail and merging to be blocked.

All of this helps us move in the direction of opening up our developmental processes to the community, but for the time being, we will keep running the internal CI as well, as it provides us with a few things which the public one doesn't (yet), including:

1. The ability to download the logs of the build ([contributions welcome](https://github.com/linkedin/venice/issues/178)!)
    
2. Some reports on which tests are most flaky, which we use to drive continuous improvements in test quality.
    
Besides the test suite, we also have a release certification environment that we use to run other kinds of tests, including performance, longevity and compatibility tests. This is not open source yet as it is tied to certain internal pieces of infrastructure, but we might be able to make a version of it public eventually.

Please let us know if you have any feedback or suggestions, or if you'd like to contribute in any way, whether to the CI or to Venice itself.
