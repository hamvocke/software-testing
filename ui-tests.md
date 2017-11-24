# UI Tests
Most applications have some sort of user interface. Typically we're talking about a web interface in the context of web applications. People often forget that a REST API or a command line interface is as much of a user interface as a fancy web user interface.

_UI tests_ test that the user interface of your application works correctly. User input should trigger the right actions, data should be presented to the user, the UI state should change as expected.

![user interface tests](img/ui_tests.png)

UI Tests and end-to-end tests are sometimes (as in Mark Cohn's case) said to be the same thing. For me this conflates two things that are not _necessarily_ related.

Yes, testing your application end-to-end often means driving your tests through the user interface. The inverse, however, is not true.

Testing your user interface doesn't have to be done in an end-to-end fashion. Depending on the technology you use, testing your user interface can be as simple as writing some unit tests for your frontend javascript code with your backend stubbed out.

With traditional web applications testing the user interface can be achieved with tools like [Selenium](http://docs.seleniumhq.org/). If you consider a REST API to be your user interface you should have everything you need by writing proper integration tests around your API.

With web interfaces there's multiple aspects that you probably want to test around your UI: behaviour, layout, usability or adherence to your corporate design are only a few.

Fortunately, testing the **behaviour** of your user interface is pretty simple. You click here, enter data there and want the state of the user interface to change accordingly. Modern single page application frameworks ([react](https://facebook.github.io/react/), [vue.js](https://vuejs.org/), [Angular](https://angular.io/) and the like) often come with their own tools and helpers that allow you to thorougly test these interactions in a pretty low-level (unit test) fashion. Even if you roll your own frontend implementation using vanilla javascript you can use your regular testing tools like [Jasmine](https://jasmine.github.io/) or [Mocha](http://mochajs.org/). With a more traditional, server-side rendered application, Selenium-based tests will be your best choice.

Testing that your web application's **layout** remains intact is a little harder. Depending on your application and your users' needs you may want to make sure that code changes don't break the website's layout by accident.

The problem is that computers are notoriously bad at checking if something "looks good" (maybe some clever machine learning algorithm can change that in the future).

There are some tools to try if you want to automatically check your web application's design in your build pipeline. Most of these tools utilize Selenium to open your web application in different browsers and formats, take screenshots and compare these to previously taken screenshots. If the old and new screenshots differ in an unexpected way, the tool will let you know.

[Galen](http://galenframework.com/) is one of these tools. But even rolling your own solution isn't too hard if you have special requirements. Some teams I've worked with built [lineup](https://github.com/otto-de/lineup) and its Java-based cousin [jlineup](https://github.com/otto-de/jlineup) to achieve something similar. Both tools take the same Selenium-based approach I described before.

Once you want to test for **usability** and a "looks good" factor you leave the realms of automated testing. This is the area where you should rely on [exploratory testing](https://en.wikipedia.org/wiki/Exploratory_testing), usability testing (this can even be as simple as [hallway testing](https://en.wikipedia.org/wiki/Usability_testing#Hallway_testing)) and showcases with your users to see if they like using your product and can use all features without getting frustrated or annoyed.
