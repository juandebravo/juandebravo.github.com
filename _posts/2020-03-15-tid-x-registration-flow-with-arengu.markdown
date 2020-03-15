---
layout: post
title: "TID-X registration flow with Arengu"
categories: [flows, arengu]
---

Last week I was working on the [TID-X](https://www.tid-x.com) registration form; TID-X is a tech event
I co-organize with my friends [Gustavo](https://twitter.com/anarchyco) and [Alonso](https://twitter.com/alalga).

Given that [TID-X](https://www.tid-x.com) is a free event, registration is a very simple process: we
need to know approximate number of attendees, so we make sure we that venu capacity is not overflowed.

### Registration flow

Despite its simplicity, the registration needs to include the following:

* **A list of all attendees including their ID details**. All that is required by the security team at the hosting venue. So, we need to make sure that we collect
all data without disclosing private information.

* **A list of attendees for the conference website**. TID-X is, mainly, a social
event focused on socializing with friends and colleagues but also meeting new
people, so getting to know who will be attending in advance is not only a cool
thing but also a useful one, if you want to make the most of your time there.

2020 will be the third TID-X edition, and in the previous ones, we (the
organizers) were tackled these requirements using two separate flows:

- *private list for security*: attendee should fill in her data in a Google Form.
- *public list of attendees*: attendee should send a Pull Request to TID-X repository including her name and personal URL.

Having two decoupled steps is the last thing you want for something like a registration flow :blush:. Also, the Pull Request step created some friction
for some attendees.

### Welcome Arengu!

I was looking for options to merge these two flows into a single step, to
simplify the usability and streamline the signing up for our users. [TID-X web page](https://github.com/tid-x/tid-x/) is hosted in Github using [Github Pages](https://pages.github.com/), and keeping the public list of attendees as part of the source code was definitely something I wanted to maintain (we're serverless!).

And then, I found **Arengu**!

[Arengu](https://www.arengu.com/) is a very interesting product that raised €500.000](https://tech.eu/brief/arengu-pre-seed/)
some weeks ago. Its goal is to help companies build their sign-up flows with ease. I'd define it as the **intersection of
[Auth0](https://auth0.com/) and [Typeform](https://www.typeform.com/)**.

Arengu has two main entities:

- *form*: usual functionalities you may expect if you've used Google Forms or TypeForm.
- *flows*: build business logic on top of a form.

Going back to the our TID-X needs, *list of attendees for the security team* was very easy to implement. With Arengu you can view and export the form submissions.

![Arengu form view submissions](/gfx/posts/arengu-form.png)

As for the *public list of attendees*, it meant updating TID-X source code by adding the new attendee to the attendees list.

That's a very good fit for an Arengu flow: once the user has filled in the form, if she has accepted to be included in the attendees public list, a new pull request to the source code repository with the relevant user data would be generated:

![Arengu flow to open a Pull Request](/gfx/posts/arengu-flow.png)

Setting up a flow (business logic) was very straight forward. The HTTP Request step triggers a
[Github Action](https://github.com/tid-x/tid-x/blob/master/.github/workflows/a%C3%B1adir-asistente.yml), which will eventually
create a Pull Request for adding the attendee name to the [attendees list](https://www.tid-x.com/inscripcion).


### Conclusion

[Arengu](https://www.arengu.com/) allowed us to merge the two registration flows we had without writing any code and without deploying any logic. Very cool! 🙌🏻🚀
