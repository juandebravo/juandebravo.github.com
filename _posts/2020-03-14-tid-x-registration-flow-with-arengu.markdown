---
layout: post
title: "TID-X registration flow with Arengu"
categories: [flows, arengu]
---

Some days ago I was working on the [TID-X](https://www.tid-x.com) registration flow; TID-X is a tech event
I co-organize with my friends [Gustavo](https://twitter.com/anarchyco) and [Alonso](https://twitter.com/alalga).

As [TID-X](https://www.tid-x.com) is a free event, the registration flow is a very simple process: we 
need to know approx the number of attendees, to make sure we don't exceed the capacity of the venue.

### Registration flow

Nevertheless, for the registration flow we have a couple of requirements:

* a requirement from the security team at the venue that holds the event: **list of attendees, including their ID**. This is private information that should not be disclosed, just sent to the security team.

* a requirement for TID-X website: **list of attendees**, but only those attendees that agreed on being part of that list. As TID-X is an event primary focused on socializing with friends, colleagues and also meeting new people, knowing who will attend is a cool thing.

In the previous two editions, we were tackling these requirements using two separate flows:

- *private list for security*: attendee should fill in her data in a Google Form.
- *public list of attendees*: attendee should send a Pull Request to TID-X repository including her name and personal URL.

Having two decoupled steps is the last thing you want for something like a registration flow :blush:. Also, the Pull Request
step created some friction for some attendees.

### Welcome Arengu!

I was looking for options to merge these two flows into a single step, so users don't need to bother about signing up in
two different places. [TID-X web page](https://github.com/tid-x/tid-x/) is hosted in Github using [Github Pages](https://pages.github.com/), and keeping the public list of attendees as part of the source code was definitely something I wanted to maintain (we're serverless!).

And I found **Arengu**!

[Arengu](https://www.arengu.com/) is a very interesting product that raised €500.000](https://tech.eu/brief/arengu-pre-seed/)
some weeks ago. Its goal is to help companies building sign-up flows with ease. I'd define it as the **intersection of 
[Auth0](https://auth0.com/) and [Typeform](https://www.typeform.com/)**.

Arengu has two different entities:

- *form*: usual functionalities you may expect if you've used Google Forms or TypeForm.
- *flows*: build business logic on top of a form.

First requirement, *list of attendees for the security team*, was very easy to implement. With Arengu you can view and export
the form submissions.

![Arengu form view submissions](/gfx/posts/arengu-form.png)

Second requirement, *public list of attendees*, meant updating TID-X source code by adding the new attendee to the attendees list.

That's a very good fit for an Arengu flow: once the user has filled in the form, if she has accepted to be included in the attendees public list, a new pull request to the source code repository would be opened:

![Arengu flow to open a Pull Request](/gfx/posts/arengu-flow.png)

Setting up a flow (business logic) was very straight forward. The HTTP Request step triggers a 
[Github Action](https://github.com/tid-x/tid-x/blob/master/.github/workflows/a%C3%B1adir-asistente.yml), which will eventually
create a Pull Request for adding the attendee name to the [attendees list](https://www.tid-x.com/inscripcion).


### Conclusion

With [Arengu](https://www.arengu.com/), we managed to merge the two registration flows we used to have without writing any code and without deploying any logic.
