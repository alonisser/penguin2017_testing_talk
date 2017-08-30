class: center, middle

# functional testing

##Or: "public app api driven tests"

###why and how?

[live presentation](https://alonisser.github.io/functional-testing-talk-penguin2017) <br/>

[twitter](alonisser@twitter.com), [medium](https://medium.com/@alonisser/), 
[sad frontend il](http://sadfrontendil.tumblr.com/)
#####Recommanded! also read my political blog: [דגל אדום](degeladom@wordpress.com)
---

# TDD in one sentence 

* Test Driven Development
* Red green refactor
* Please note: This does not say: Do only unit and unit are better

---

# TDD revisited

* Lots of over zealous people attacking 
* Test "first" is not the same as "must test"
 
---

# Why tests?

* Validate our code is actually doing what we expect.
* Promotes good (better..) design, clearer api, decoupling.
* Faster development cycles (Really!) catching bugs early and fast.
* Ability to refactor (even a year later when the code does not look familiar anymore)
* Ours users are not our QA "volunteers" .


---
# The traditional view: The Test Pyramid

* "Unit is fast" use unit
* "DB is hot lava" avoid touching the real db in your tests at all cost.
* Higher level tests are slow and brittle, don't do that, at least not too much.

---

# The traditional view: The Test Pyramid

![](https://martinfowler.com/bliki/images/testPyramid/test-pyramid.png)

---


# The traditional view: Unit test

* "Unit is fast" use unit
* Thousands of unit tests in few seconds
* low-level, focusing on a small part of the software system.
* Lots of Unit testing practitioners *nowadays* advocate non sociable tests with mocking collaborating classes or even private methods.
* low-level, focusing on a small part of the software system. 

---

# The traditional view revisited: Problems With UI driven testing

Selenium and similar tools:
* SLOW! 
* BRITTLE. Broken easily on UI changes (Even with best practice patterns: page objects etc)
* FLAKY. 5-10% failure due to "network" or "browser" conditions renders the tests (almost) worthless. 

---

# The traditional view revisited: Problems With Unit testing

* The trees and forest problem.
* Too much mocking is dangerous, we can break the class but the tests still pass.
* Passing unit test [don't validate](https://twitter.com/thepracticaldev/status/845638950517706752?lang=en) the problem domain solved. 
* Might lead to tight coupling of tests to implementation.
* Blamed for design monstrosities: "Test-induced design damage", needed to abstract all db and IO code out of tested classes. 

---

# The traditional view revisited: The Test Pyramid
An often missed side note:

"
    The pyramid is based on the assumption that broad-stack tests are expensive, slow, and brittle compared to more focused tests, 
    such as unit tests. While this is usually true, there are exceptions. If my high level tests are fast, reliable, 
    and cheap to modify - then lower-level tests aren't needed.

"

---

# The traditional view revisited: functional testing - the new unit?

* In an Interesting twitter "war" not long ago, @DHH (of Rails fame) claimed that the "DB is part of your app now" and DB hitting tests are the new unit.
--
* When we are using DB "features" like Mongo GIS or Redis timeseries, then mocking the DB makes the tests faster but we don't really test the feature
* With a proper testing support in the framework, DB setup/rollback can be very fast (Like ROR testing over a transaction)
--
* Http is a leaking abstraction. 
 
---

# Naming issues

functional vs Integration vs system vs .. tests
* Looks like everyone "knows" (or has an opinion) what is a "unit" but the rest is a complete mess
--
* **MY** definition for functional tests are tests using the app public api (http/socket/Message queue/ what ever), "beneath" the UI.
--
* They cover a **single** app/service, not inter app/service integration (That would be under **End To End** or **Integration** testing)
* Google testing blog suggests a test "size" taxonomy: small, medium. I'm equating "medium" tests to "functional" tests
* We can also use Martin Fowler ["subcutaneoustest"](https://martinfowler.com/bliki/SubcutaneousTest.html) if we can pronounce it.

---
# Naming issues

Even more confusion 
* UI Testing isn't the same thing as End to End testing
* UI Testing isn't the same as Higher level testing.
--
* UI components encapsulating logic can and should be tested using unit testing.

---

# Functional testing: High level overview

* Testing features! not code constructs like classes or controllers.
* Testing the public Api of a specific app. Not testing internals (Mostly)
* Driving the tests through the app/service api, no internal mocking, replacing Only third party collaborators.

---

# Functional testing patterns by example

* Meaningful Naming
 
---

# Functional testing patterns by example

* verify zero state
 
---

# Functional testing patterns by example

* Terse and readable tests.
* Abstract technical test code to helper methods/classes

---

# Functional testing patterns by example

* BDD style methods (http://martinfowler.com/bliki/GivenWhenThen.html)

---

# Functional testing patterns by example: Setup and data creation

* The problem with fixtures: Another thing we need to update

---

# Functional testing patterns: Setup and data creation

* data builders using the app api
 
---
class: center, middle

# Anti Patterns

* An Intermezzo
 
---
# CI: Does A test exists if it does not run?

* "Sure we have automatic tests"
* Wiring up a Continues Integration process is a build in part of writing tests. 
* If you don't have one, **You don't really have tests**
 
---


# The commented out test anti pattern

* "Sure we have automatic tests"
--
* OH
* Remember: A flaky test is usually points to a problem in the test, BUT sometimes.. It serves as a warning from a subtle race condition, implicit order bug, etc
--
* Another one are [Assertion Free tests](https://martinfowler.com/bliki/AssertionFreeTesting.html)
 
---

# Functional testing patterns: Replacing third party collaborators with fake collaborators
* Wrapping third party collaborators in proxies. 
* Proxies should not used directly but "injected" into our code
Instead of this:

```python
class SMSDistributionHandler(BaseDistributionHandler):
    
    def __init__(self):
        self.sms_sender = SMSSendingService()

```

Do this (if you don't have a better DI framework and not passing collaborators through CTOR  :)  ):
```python
class SMSDistributionHandler(BaseDistributionHandler):
    
    def __init__(self):
        self.sms_sender = ServiceLocator.get_service('sms_sender')
```

* Similar to the way Django allows us to replace some backends with fake backends in settings.py

---

# Functional testing patterns: Asserting against fake collaborators
```python

class TestApp(object):

    def verify_sms_has_not_been_sent(self, message, receiver_phone, sender_phone):
        sms_messages = self._get_sms_by_predicate(lambda x: x.message == message and x.sender == sender_phone and
                                                            x.recipient == receiver_phone)
        self.assertFalse(len(sms_messages))

    def _get_sms_by_predicate(self, predicate):
        return query(self.sms_service.messages).where(predicate).to_list()

```

---

# Functional testing patterns: Globals

Globals are also collaborators that you might want to control

* Time

* Logging

(See GOOSG for more examples)

---
# Functional testing patterns: Globals example - Handling global time

* No magical or implicit datetime/time handling in models (no auto_add_now)
* use a programmable wrapper over ```datetime.now()```

```python
# -*- coding: utf-8 -*
from django.utils import timezone

time_settings = {
    'frozen_time': False

}

def freeze_time(frozen_time):
    time_settings['frozen_time'] = frozen_time

def unfreeze_time():
    time_settings['frozen_time'] = None

def now():
    if time_settings['frozen_time']:
        return time_settings['frozen_time']
    return timezone.now()

```
And in our our actual module
```python
import clock
clock.now()
```
---
# Functional testing patterns: Handling async results with a trace

* 


---
# When do I unit? 

* Certainly not advocating against unit testing. sometimes unit testing is more valuable/thorough and or quicker.
For example:
    * A component encapsulating logic: a rule engine or a calculator class for example.
    Especially when there are lot's of decisions and possible permutations, functional testing this would be painful.
    * A UI component with inner logic mapped to presentation: React/Angular/vue.js component, django template tag
    * "Simple" helpers: A formatting helper for example, Id normalizer, etc. 

Generally speaking: Pure functional code encapsulated within a class/function, without IO, is a good candidate for unit testing
---

# When do I unit?

* THIS presentation is about apps, or functional microservices. If you are writing a library then Unit testing is probably more suited for your needs.


---

# What you should take from this talk

* Test you app. Please. 
* Treat testing as an integral part of coding your app.
* You'll thank me for this

---

# Testing taxonomy 

* http://martinfowler.com/bliki/subcutaneoustest.html
* http://martinfowler.com/bliki/BroadStackTest.html
* [Google testing blog - on testing sizes](https://testing.googleblog.com/2010/12/test-sizes.html)
---

# interesting resources and further reading

* [GOOSG book](http://www.growing-object-oriented-software.com/)
* [Against unit tests](http://www.obeythetestinggoat.com/fast-tests-useless-hot-lava-be-damned.html)
* [boundaries - A great must watch talk by Gary Bernhardt which goes against a lot I talked about here](https://www.destroyallsoftware.com/talks/boundaries)
* [Give when then testing style](http://martinfowler.com/bliki/GivenWhenThen.html)
* [The test pyramid](http://martinfowler.com/bliki/TestPyramid.html)
* [TDD is Dead. long live testing/@DHH](http://david.heinemeierhansson.com/2014/tdd-is-dead-long-live-testing.html)
* [Is TDD Dead? conversions between Kent Beck, David Heinemeier Hansson, and Martin fowler](https://martinfowler.com/articles/is-tdd-dead/)
* [Google testing blog - against End To End tests](https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html)
* [Working End TO End tests](https://www.symphonious.net/2015/04/30/making-end-to-end-tests-work/)

---

class: center, middle

# open source rocks!
## thanks for listening!

[live presentation](https://alonisser.github.io/functional-testing-talk) <br/>
[twitter](alonisser@twitter.com), [medium](https://medium.com/@alonisser/), 
[sad frontend il](http://sadfrontendil.tumblr.com/)

---
