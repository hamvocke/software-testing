# The Importance of (Test) Automation
Software has become an essential part of the world we live in. It has outgrown it's early sole purpose of making businesses more efficient. Today companies try to find ways to become first-class digital companies. As users everyone of us interacts with an ever-increasing amount of software every day. The wheels of innovation are turning faster.

If you want to keep pace you'll have to look into ways to deliver your software faster without sacrificing its quality. **Continuous delivery**, a practice where you automatically ensure that your software can be released into production any time, can help you with that. With continuous delivery you use a **build pipeline** to automatically test your software and deploy it to your testing and production environments.

Building, testing and deploying an ever-increasing amount of software manually soon becomes impossible -- unless you want to spend all your time with manual, repetitive work instead of delivering working software. Automating everything -- from build to tests, deployment and infrastructure -- is your only way forward.

![build pipeline](img/buildPipeline.png)

*Use build pipelines to automatically and reliably get your software into production*

Traditionally software testing was overly manual work done by deploying your application to a test environment and then performing some black-box style testing e.g. by clicking through your user interface to see if anything's broken.

It's obvious that testing all changes manually is time-consuming, repetitive and tedious. Repetitive is boring, boring leads to mistakes and makes you look for a different job by the end of the week.

Luckily there's a remedy for repetitive tasks: **automation**.

Automating your tests can be a big game changer in your life as a software developer. Automate your tests and you no longer have to mindlessly follow click protocols in order to check if your software still works correctly. Automate your tests and you can change your codebase without batting an eye. If you've ever tried doing a large-scale refactoring without a proper test suite I bet you know what a terrifying experience this can be. How would you know if you accidentally broke stuff along the way? Well, you click through all your manual test cases, that's how. But let's be honest: do you really enjoy that? How about making even large-scale changes and knowing whether you broke stuff within seconds while taking a nice sip of coffee? Sounds more enjoyable if you ask me.
