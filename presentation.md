class: center, middle

# functional testing

##Or: "public app api driven tests"

###why and how?

[live presentation](https://alonisser.github.io/penguin2017_testing_talk) <br/>

[twitter](alonisser@twitter.com), [medium](https://medium.com/@alonisser/), 
[sad frontend il](http://sadfrontendil.tumblr.com/)

####Recommanded! also read my political blog: [דגל אדום](degeladom@wordpress.com)
---

# TDD in one sentence 

* Test Driven Development

--

* Red green refactor

--

* Please note: This does not say: Do only unit and unit are better

---

# TDD revisited

* Lots of over zealous people fighting about "pure" TDD
 
* A lot has changed on how we build software

* Still lots and lots of untested new code is written every day. why?
 
---

# Why tests?

* Validate our code is actually doing what we expect.

* Promotes good (better..) design, clearer api, decoupling.

* Faster development cycles (Really!) catching bugs early and fast.

* Ability to refactor (even a year later when the code does not look familiar anymore)

* Ours users are not our QA "volunteers" :( .

---

# The traditional view: The Test Pyramid

![](https://martinfowler.com/bliki/images/testPyramid/test-pyramid.png)



---
# The traditional view: The Test Pyramid

* "Unit is fast" use unit

* "DB is hot lava" avoid touching the real db in your tests at all cost.

* Higher level tests are slow and brittle, don't do that, at least not too much.

---

# The traditional view: Unit test

* "Unit is fast" use unit

* Thousands of unit tests in few seconds

* low-level, focusing on a small part of the software system.

* Lots of Unit testing practitioners *nowadays* advocate **unit tests as non sociable (Isolated) tests with mocking collaborating classes or even private methods**.

---

# The traditional view revisited: Problems With UI driven testing

Selenium and similar tools:
* SLOW! 

* BRITTLE. Broken easily on UI changes (Even with best practice patterns: page objects etc)

* FLAKY. 5-10% failure due to "network" or "browser" conditions renders the tests (almost) worthless. 

---

# The traditional view revisited: Problems With Unit testing

* The trees and forest problem. Is the feature actually working?

* Too much mocking is dangerous, we can break the class but the tests still pass.

* Passing unit tests [doesn't validate](https://twitter.com/thepracticaldev/status/845638950517706752?lang=en) the problem domain solved.
 
* Might lead to tight coupling of tests to implementation.

* Blamed for design monstrosities: "Test-induced design damage", needed to abstract all db and IO code out of tested classes. 

* Following the previous point: To really test every part of our app with unit test we might need to give up on lot's of what Modern Web Framework give us (Django, Ror, etc)
---

# The traditional view revisited: The Test Pyramid
An often missed side note:



>*The pyramid is based on the assumption that broad-stack tests are expensive, slow, and brittle compared to more focused tests, such as unit tests.
 While this is usually true, there are exceptions.
 If my high level tests are fast, reliable,  and cheap to modify - then lower-level tests aren't needed.*




--
## So. Can we write high level tests which are (relatively) fast, reliable and cheap to modify?


---

# Functional Api driven testing: definitions

functional vs Integration vs system vs .. tests
* Looks like everyone "knows" (or has an opinion) what is a "unit" but the rest is a complete mess
--
* **MY** definition for functional tests are:

## Public Api driven testing for the app features from "outside" . 

--
* Google testing blog suggests a test "size" taxonomy: small, medium. I'm equating "medium" tests to "functional" tests

* We can also use Martin Fowler ["subcutaneoustest"](https://martinfowler.com/bliki/SubcutaneousTest.html) if we can pronounce it.

---

# Functional Api driven testing: definitions

* Using the public api endpoints (http/socket/management commands/Message queue/ what ever).

* Covering a **single** app/service, not inter app/service integration (That would be under **End To End** or **System** testing)

--

* What it isn't: It's **not** about automated "UI clicking"

---

# functional testing - the new unit?

* In an Interesting twitter "war" not long ago, @DHH (of Rails fame) claimed that the "DB is part of your app now" and DB hitting tests are the new unit.

--

* When we are using DB "baked in features" like Mongo GIS or Redis timeseries, The Db becomes "part" of the actual unit under test.

--
* SLOW? With a proper testing support in the framework, DB setup/rollback can be very fast (Like ROR testing over a transaction) Also parallel computing can go a long way in solving this kind of problems

* SLOW? the tests might be a slower but developing with Modern Web framework is Faster then giving up on them. Which speed is more important? 
Functional testing might be easier to get quick testing gains

--

* Software has leaking abstractions. (Http for example)  
 
---

# Functional testing: High level overview

* Testing features! not code constructs like classes or controllers.

* Testing the public Api of a specific app. Not testing internals (Mostly)

* Driving the tests through the app/service api, no internal mocking, replacing Only third party collaborators.

---

# Functional testing patterns by example

* Meaningful Naming

.code-example.pull-left[
```python
#bad
def test_save(self):
    # Test stuff
```
]

.code-example.pull-right[
```python
#better
def test_new_expense_creation(self):
    # Test stuff
```
]

 
---

# Functional testing patterns by example

* verify zero state

.code-example.pull-left[
```python
#bad
def test_new_expense(self):
    self.when_something_happens()
    assert len(expenses) > 1
```
]

.code-example.pull-right[
```python
#better
def test_new_expense(self):
    assert not len(expenses) > 1
    
    self.when_something_happens()
    assert len(expenses) > 1
```
]

---


# Functional testing patterns by example

* [GivenWhenThen](http://martinfowler.com/bliki/GivenWhenThen.html) style methods 
--

* Given: setup that is before running action we are testing

* When: what we do - usually what we test

* Then: following verifications

.code-example[
```python
#bad
def test_new_expense_reporting(self):
    self.client.post('my_supplier')
    
    res = self.client.post('my_expenses_endpoint', {var1:param1, var2:params2})
    
    expenses = Expense.objects.filter(supplier=param2)
    assert len(expenses) > 0 
```
]

.code-example[
```python
#better
def test_new_expense_reporting(self):
    self.verify_no_expenses_have_been_reported()
    
    supplier = self.given_the_supplier_exists(name)
    
    self.when_user_reports_an_expense(price, supplier)
    
    self.verify_new_expense_has_been_reported()
```
]

---

# Functional testing patterns by example

* Terse and readable tests.

* Abstract technical test code to helper methods/classes

* Use the Domain language

.code-example[
```python
#bad
def test_new_expense(self):
    res = self.client.get('my_expenses_endpoint?supplier=kravitz')
    
    assert res.ok
    expenses = res.json()['data']['expenses']
    assert len(expenses) > 1
```
]

.code-example[
```python
#better
def test_new_expense(self):
    
    self.verify_expense_has_been_reported(supplier_name=kravitz)
```
]

---

# Functional testing patterns by example: Setup and data creation

* The problem with fixtures:  Yet Another thing we need to update 
* Might work for simple, non changing configurations but Don't scale well with complexity
---

# Functional testing patterns by example: Setup and data creation

* Solution: data builders encapsulating the app api

```python
person = PersonTestBuilder(name=alon)
            .set_question(42)
            .add_wish("The one ring")

# the builder allows optional complex setup while still readable.
```

* Methods should still use the app public api 
 
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

class: center, middle

# Advanced testing Patterns

---
# Functional testing patterns: Third party collaborators

* Replacing third party collaborators with fake collaborators

.code-example[
```python
class SmsSenderProxy(ISmsSenderProxy):
    # Real proxy, handles logic via third party collaborator
    def send(message):
        
        return requests.post(data)
```        
]        
        
.code-example[
```python
class FakeSmsSenderProxy(ISmsSenderProxy):
    # Replacement proxy
    def __init__(self):
        self.messages = []
        
    def send(self, message):
        return requests.post(data)      
```
]
---
# Functional testing patterns: Replacing third party collaborators with fake collaborators
* Wrapping third party collaborators in proxies.
 
* Proxies should not used directly but "injected" into our code

.code-example[
```python
# instead of this:
class SMSDistributionHandler(BaseDistributionHandler):
    
    def __init__(self):
        self.sms_sender = SMSSendingService()

```
]
.code-example[

```python
# Do this (if you don't have a better DI framework and not passing collaborators through constructor  :)  ):
class SMSDistributionHandler(BaseDistributionHandler):
    
    def __init__(self):
        self.sms_sender = ServiceLocator.get_service('sms_sender')
```
]

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

Ever exprienced a test failing between 12:00 - 02:00? 

--

Globals are also collaborators that you might need to control. For example:

* Time

* Logging

(See GOOSG for more examples)

---

# Functional testing patterns: Handling global time

* No magical or implicit datetime/time handling in models (no auto_add_now)
* use a programmable wrapper over <code>datetime.now()</code>

```python
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
---

# Functional testing patterns: Handling global time

* And in our our actual module

```python
import clock
clock.now()
```

Or if we need to manipulate time
```python
clock.freeze_time(datetime.datetime(2017, 7, 12, 30. tzinfo=pytz.UTC)
```

---

# Functional testing patterns: Handling async results with a trace

* How to assert against the result of async process "going out" - sending stuff etc..?

--

* Setup a trace that would receive outgoing messages in tests and check for expected result in recurring intervals

---

# Functional testing patterns: Handling async results with a trace

 
```javascript

class Trace {
  constructor() {
    this._trace = []
  }
  add(item) {
    this._trace.push(item);
  }
  contains(predicate, count, deferred, timer) {
    var deferred = deferred || defer();
    var timer = timer || 1;
    var count = count ? count : 1;
    var results = _.filter(this._trace,  (item) =>{
      return predicate(item);
    });

    if (results.length == count) {
      deferred.resolve(true);
    } else {
      if (timer < 10) {
        timer++;
        setTimeout( () =>{
          return this.contains(predicate, count, deferred, timer)
        }, 250);
      } else {
        deferred.reject(`Current trace contains: ${JSON.stringify(this._trace)}`);
      }
    }
    return deferred.promise;
  }
 }
```

---

# When do I unit? 

* Certainly not advocating against unit testing. sometimes unit testing is more valuable/thorough and or quicker.

--

Good unit candidates:

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

* Test you apps. Please.
 
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

# Questions?

---

class: center, middle

# open source rocks!
## thanks for listening!

[live presentation](https://alonisser.github.io/penguin2017_testing_talk) <br/>
[twitter](alonisser@twitter.com), [medium](https://medium.com/@alonisser/), 
[sad frontend il](http://sadfrontendil.tumblr.com/)

---
