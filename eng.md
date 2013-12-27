Backbone.js deserves a lot of credit for bringing MVC to mainstream client-side Javascript development. That said, many beginners ask what the ‘right way’ of doing something with Backbone is. The bad news is that there’s not necessarily a ‘right way’ – it all depends on the problem you are trying to solve. The good news is that there are definitely some ‘wrong ways’ that you should avoid on your way to finding the right solution for your particular problem.

In this post I’ll cover some of these anti-patterns, as well as some general advice for the new starter. I’ve ordered the anti-patterns roughly by significance from the major to the more trivial. Don’t be too upset if you’ve done something on this list – I’ve made most of these mistakes myself ;)


Antipattern #1: Building a single page app when you don’t need to

This first one doesn’t really pertain to Backbone specifically, but it’s worth mentioning up-front: If you’ve decided to build a single-page app, make sure you’ve got a pretty good reason to do so.

At first glance, the concept can seem appealing: instead of having to assemble fragments of HTML and Javascript on the server-side, just define a JSON API and do everything on the client-side with Javascript. However, there’s a couple of things you need to be wary of.

Firstly, if your app is using some sort of database as the backend (highly likely), a JSON API is effectively just another layer of indirection between your UI code and that database. From a development perspective, it’s much easier and more flexible for UI code to live on the server and be able to make direct SQL calls to your database rather than have to define and then implement a JSON API. 

Secondly, building a single-page app is just plain hard, especially if your background is in building traditional request-response apps with frameworks like PHP or Rails. SPAs are all asynchronous and event-based, and objects can live for a long time. This is very different from the synchronous, short-lifecycle approach of old-school web apps.

Finally, it’s worth noting that just because you’re using Backbone (or Angular, or many other Javascript MVC frameworks) you don’t have to commit to building a full single-page app. Perhaps you can just use it for those parts of your app where the UX demands it. For example, 37Signals implemented the new Basecamp calendar with Backbone, without making all of Basecamp a single-page app.

Antipattern #2: Using Backbone When You Should Use Another Framework

The Javascript MVC framework you choose for a project is one of the most important decisions you’ll make, and Backbone is by no means the best choice for all projects. For more information check out my prior post on why Backbone Is Not Enough for many Javascript-intensive applications.

Antipattern #3: No View Tests

Backbone Models are reasonably straightforward to unit test. Backbone Views can be a little more tricky – but this doesn’t mean you shouldn’t do it.

In addition to the well-understood benefits of unit testing (i.e., code quality), writing unit tests for your views forces you to think about how your code is structured. Because it is so easy to set up interdependencies between views via global variables and undocumented properties, un-tested code will tend gravitate towards a state where there are no clearly defined units of code. This is a bad thing.

For more information on testing Backbone Views, check out my previous post on Testing Backbone Views with QUnit and Sinon.

Antipattern #4: No Memory Management

A classic Backbone beginners mistake is to not think about memory management. As a consequence, in no time at all an app can be leaking memory like a sieve and triggering a blizzard of events whenever anything happens.

‘How can this be?’, the novice ask, ‘my code is super tight and Javascript is supposed to be garbage-collected’. Well unfortunately there’s one source of memory leaks that Javascript can’t pick up: views that continue to listen to their models for events, even when those views are no longer being displayed. Views stay in memory because models still retain references to them, and this pool of ‘zombie views’ grows over time.

Fortunately this is a well-understood problem, for which there are a variety of solutions. Whatever you do, an important first step is to use Backbone’s listenTo method when subscribing to events rather than using the ‘on/bind’ methods directly. At least that way, the subscriber can keep a list of what its listening to. However, it’s still up to you (or your view-management framework – we often use Backbone LayoutManager) to get your views to stop listening to events when it is no longer being used.

Antipattern #5: Data Attributes in the DOM

The next anti-pattern is probably the most common thing I see those new to Backbone do – particularly if they’re coming from a jQuery background. It’s probably best demonstrated with an example.

Say that we want to display a list of people, and respond when an item in the list is clicked. The models will look something like this:


What’s the problem with this? Well, inserting data into the DOM and then pulling it out again kind of runs against the grain of Backbone and tends to box you in as your UI gets more complex.

Instead, we’d be better-off leveraging everything Backbone gives us for associating portions of our DOM with data. Given that there’s a one-to-one correspondence between each Person model and their <li> row in the DOM, we might as well capitalise on that by having a view that mediates between the two of them.

Let’s change our people template to be:


and create another template called dedicated to displaying a person:

Then define a view for the Person model:

and have the PeopleView render this into place:

No more data attributes, no more event targets. This also makes testing easier, as you can now unit test a PersonView in isolation.

Put bluntly, don’t be shy about having lots of views. To be honest, it’s very rare for me to use iterators in templates, unless I’m rendering purely read-only data. I consider data attributes to be a code smell that will cause problems down the track.

Antipattern #6: Rendering Templates Asynchronously

Frameworks like Backbone.LayoutManager make it easy to load templates asynchronously. However, it’s easy to underestimate how much asynchronicity complicates both your code and your tests – it should be avoided unless absolutely necessary. And a lot of the time, asynchronous template loading is unnecessary.

This is because in production environments it’s unlikely that you’ll be loading templates individually across the network – they’ll probably be precompiled and concatenated into your application Javascript file. Consequently, they’ll never actually be be loaded asynchronously.

This leaves your local development environment as the only place that you’d want to load templates asynchronously. But funnily enough, if you switched to synchronous loading for local development, you probably won’t notice much in the way of a performance degradation, as all your synchronous calls will just local.

So this eliminates the only rationale for loading templates asynchronously.  So why do it at all? For many of my projects, I’ve switched to using the ‘async: false’ option in the Ajax calls that fetch templates in dev. My code was able to become much simpler, and things didn’t get noticeably slower during dev.

If you don’t want to compile all your code into a single artefact, you can still break it into chunks and load those separately. However, each chunk should comprise both the code _and_ compiled templates relevant to a section of your app. Loading templates as they are needed is probably too fine-grained an optimisation and will make your app excessively chatty, not to mention harder to code.

Antipattern #7: Undocumented Options

Backbone Views can be initialized with an ‘options’ object, which can contain just about anything. The options object is extremely handy, because it means you can pass anything into a view via options. However, it’s also very easy to abuse, because options don’t have to be declared anywhere (for example, as arguments to the initialize() method).

In earlier versions of Backbone, this object would automatically be bound to this.options. In Backbone 1.1, the situation improved slightly because the automatic binding doesn’t happen anymore. However, it’s still easy for somebody to do the binding themselves.

Either way, the consequence is that if you didn’t write the code, you often only find out about an option when it got used deep within a class. For example:

Nobody will know that the view has a dependency on an obscure backdoor until they find the options reference deep within the code.

This is unfortunate because each option object provided to a view is effectively a coupling between the view and that object. Put differently, the options that a view takes constitute part of the public interface of that view.

Consequently, I highly recommend that all options that a model or view uses be documented somewhere in the class or initializer declaration (if there is one). It can look as simple as this:


If you’re feeling particularly diligent, you can document which options are mandatory and which aren’t actually required. Or you can even check for the presence of mandatory options in the initialize() method of your object. Either way, if you don’t document options, there’s a good chance you’re going to end up with spaghetti code that’s full of hidden backdoors.

Antipattern #8: Premature Use of Custom Events

It’s common for new starters to Backbone to get excited about the fact that just about everything can emit custom events. Consequently, they start to add their own custom events.

I have a couple of issues with this.

Firstly, the events that an object emits constitute a part of the public interface for that object. So if you’re going to have something emit custom events, it’s really important that you document that it can do so. Otherwise, you’re effectively adding backdoor coupling that is difficult for those unfamiliar with the code to see.

Secondly, often custom events are added under the pretence that it ‘decouples’ the code, but the fact remains that if some required behaviour is implemented by having two objects interact via custom events, then those objects are still coupled – just with an extra level of indirection.

In the case of view interaction, I personally have no issues at all with two views having references to each other – as long as those references are well-documented (see anti-pattern #7). This is simpler and easier to test.

The most useful events are those that are well-defined, globally understood, and potentially of interest in more than just one or two special-cases. The existing Backbone built-in events (http://backbonejs.org/#Events-catalog) meet these criteria. They can take you a long way if you fully leverage them.

For example, one strategy that maximises the use of Backbone’s built-in events is to coordinate changes between several views via the underlying models that are shared by those views. One view could trigger a change to a model, which makes an update to another model, and that change is then picked up by another view that is listening for change events.  For complex views, introducing a tailor-made view-model can also be handy, and also make testing easier. Neither of these strategies require the introduction of custom events in the first instance.

The only place I’ve found custom events to be useful is in views that you want to reuse through your apps – for example, UI components like tabs or accordions. Custom events are the more flexible way for people to detect what a component is doing so that they can customise its behaviour for a particular scenario. Just make sure that these custom events are well-documented!

Antipattern #9: Building a Relationship Mapper

On non-trivial apps it’s not uncommon to want to deserialize nested JSON data structures from your server into a tree of Backbone models and collections, then potentially re-serialize that tree (or portions of it) for display in templates or for sending back to the server.

One way to do this is to override the parse and toJSON methods on your models and do it yourself manually. However, next you want to lazily load certain portions of the tree, so you put in special logic for that. Then you get sick of duplicated code in parse and toJSON, so you try to write a mechanism for specifying your relationships declaratively instead. Then you find that an object with the same ID and type is appearing in different parts of your JSON, and you want it to be represented by the same model instead, so you need to implement some sort of identity map.

Before you know it, things are getting out of hand and you’ve found yourself writing a pseudo-object/relational mapper. This is something your should avoid at all costs – object/relational mapping is something that seems straightforward at first but is littered with nasty edge-cases.

Fortunately, there are a number of Backbone frameworks out there that do this sort of stuff for you already. Backbone-Relational is probably the most heavyweight. It’s very powerful and includes an identity map, but – like traditional O/R mappers – you can easily shoot yourself in the foot if you don’t make yourself familiar with the subtleties of how it works. Backbone-Associations and Backbone-Nested are more lightweight and may be sufficient for your needs.

Whatever you do, make sure you check them out before you do it yourself.

Antipattern #10: Redundant Divs

This is a minor one, and opinions can differ on it. However, I think it’s worth mentioning because a lot people don’t even realise it happens.

When rendering templates, remember that by default Backbone creates an empty div element for each Backbone view. This can lead to a lot of redundant divs in your generated HTML.

Here’s an example. Consider a templates like this:


So when you render the view, it looks like this:


Those redundant divs really add up as your DOM gets more complex.

The thing is that Backbone lets you specify details about the root element of a view, including what the element type should be. So instead, you could do this for the template:



And provide more details about the root element in the view:



When rendered, the view will look like this:


Much better. I used to not really understand why they’re called ‘Views’ rather than ‘Controllers’, but now I think I understand. I consider it a code smell if I see a template with a single root element. Use everything that Backbone gives you, and don’t balk at putting a little bit of view-related stuff in your View objects – that’s why they’re called ‘Views’, after all.

Some Final Advice

That sums up my top ten Backbone anti-patterns, ranging from the fundamental to the finicky. However, I appreciate that being told what not to do still leaves a very large set of possible things you could do.

So if I had to give one last piece of guiding advice to the fledgeling Backbone developer, it would be this: always start with the UX and work backwards from there. By extension, this means working backwards from your templates and Backbone Views before getting too bogged down trying to use fancy across-the-board architectural approaches. Don’t be afraid to have lots of view classes and – where necessary – link between them.

That said, you should also unit test and refactor extensively as you go. Duplicated business logic in your views? Push it down into your models. Views still having to know to much about when models are being updated? Start using event listeners on the models so the views don’t have to know in advance when they’re going to change. Views getting too big? Break them down. Views still too complicated? Introduce an intermediate layer of view-models.

A continual code/test/refactor cycle (or test/code/refactor cycle if you’re that way inclined) will usually leave you with just the right degree of design. If possible, code reviews also help a lot too. Finally, make sure that you make yourself familiar with everything that Backbone and its plugin ecosystem gives you, rather than accidentally reinventing the wheel yourself.

If you’ve got some of your own Backbone Antipatterns, I’d love to hear about them!