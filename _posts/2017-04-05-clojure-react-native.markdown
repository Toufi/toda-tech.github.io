---
layout: post
title:  "Clojure + React Native"
date:   2017-04-05 16:42:53 -0400
categories: clojure react client-app
---

We're working closely with our joint venture, TQF, on developing
native mobile clients for our ledgerless blockchain tech.  One thing
we'd like to open up about today is how we're able to develop natively
for both iOS and Android in a highly leveraged way, and some thing
unique about our engineering approach to these apps.

While we absolutely wanted to develop native applications for these
platforms in order to deliver a high-quality look-and-feel, as well as
native-grade responsiveness, we were very reluctant to split our
engineering efforts three ways (Clojure library, iOS Objective C app,
and Android Java app).

[React Native][react-native] offered a JavaScript-based solution which was appealing
to our engineering team due to its near-religous focus on a
single-point-of-truth for all application state, which eliminates a
major source of runtime errors.  It also offered an opportunity to
eliminate two rigid, static development projects and replace them with
a single effort based on a dynamic language.

Coupling [Clojure][clojure]([Script][clojure-script]) with React Native compounded these benefits
enormously. Our preferred framework for this turned out to be
[Re-Natal][renatal], as we judged it to be the most mature, and best
supported. We make extensive use of [clojure.spec][spec] to type-check the
entire state of the application.  Every change to application state is
funneled through a single mechanism, which enforces a spec validation
each time.  This way, invalid state is detected immediately, and the
application is prevented from acting on the consequences of
potentially corrupt data.  Furthermore, debugging becomes much easier,
since the offending code usually ends up in the stack trace of the
validation error.

Even modifying the state of a toggle switch is validated against the
spec at runtime:

{% highlight clojure %}
(def toggle-xfer-direction
  {:to-wallet :to-loc
   :to-loc :to-wallet})

(reg-event-db
 :xfer-toggle
 validate-spec
 (fn [db _]
   (update-in db [:loc-xfer-direction] toggle-xfer-direction)))
{% endhighlight %}

Another surprising benefit to writing native applications in
ClojureScript is the development environment, and one of its key
features, [figwheel].  Our team is able to spin up a [repl] inside emacs
and send code from buffer they're working on - which is then evaluated
inside the context of the running native application!  Given this
power, it's difficult to imagine developing a native application any
other way.

Finally, using the same language for our base library, core platform,
as well as our React Native client applications means that we are
writing code once, and running it on our main server, command-line
client, web server, iOS app and Android app: this is serious leverage.

We've had a few rough patches getting ClojureScript and React Native
to play properly together, which slowed the beginning of the dev
effort, but we've more than made up for the time.  React documentation
is very Javascript-specific, and little of it refers to Native, and
none of it to ClojureScript concerns.  We've also found that some of
the ClojureScript bindings are [a bit too opaque][opaque], or magical, and that
reading the source was the only way to really understand what was going on.  

But, ultimately, this choice has turned out to be a success - and
we're looking forward to sharing the results soon!

Adam

[spec]: https://clojure.org/about/spec
[figwheel]: https://github.com/bhauman/lein-figwheel
[react-native]: https://facebook.github.io/react-native/
[clojure-script]: https://clojurescript.org/
[clojure]: https://clojure.org/
[renatal]: https://github.com/drapanjanas/re-natal
[repl]: https://github.com/bhauman/lein-figwheel/wiki/Using-the-Figwheel-REPL-within-NRepl
[opaque]: https://github.com/reagent-project/reagent/blob/master/src/reagent/impl/template.cljs