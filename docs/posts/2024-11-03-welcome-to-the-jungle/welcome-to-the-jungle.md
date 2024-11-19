---
draft: false
date: 2024-11-03
slug: welcome-to-the-jungle
categories:
  - Philosophy
  - Leadership
---

# Welcome to the Jungle

Your coding conventions, branching strategy, and review guidelines may seem like superficial policy hygiene. But these, along with seemingly random and disconnected activities like fixing a flaky unit test, are connected and can have a profound impact on how your team works with code.

<!-- more -->

As a principal software architect and someone who cares deeply about how we develop software, I have mostly stopped writing down policies. Instead, I have started creating environments — jungles — where following a policy is the path of least resistance. By thinking about software development as a cultural ecology, you can unlock more flexible, efficient, and less bureaucratic ways of organizing software development.

If you are a senior developer, architect, tech lead, or CTO, I invite you on an excursion. We’ll travel winding paths through philosophy, biology, and social sciences to discover new ways of thinking about software development.
Developers trekking through a rainforest

![Trekking through rainforest](trekking.jpeg)


What is a software ecosystem? A global multitude of programs, libraries, and operating systems? A whirlpool of interactions between developers, teams, and organizations?

Anthropologist Julian Steward coined the term **cultural ecology** in 1955 to describe the way that *“cultural change is induced by adaptations to the environment.”* Just like a biological ecosystem evolves and changes when different lifeforms interact in complex webs, societies evolve in relation to their cultural environments.

The now famous quote from Rachel Carson’s book Silent Spring: *“In nature, nothing exists alone”*, applies to all ecologies, both biological and cultural.

Software engineering is a particular and peculiar form of cultural ecology. A varied range of lifeforms — developers, scrum teams, companies, and open source projects — interact with an environment of programming languages, CI/CD systems, and office cubicles. The environment limits some behaviors but also presents opportunities for the life forms. But the influence is bi-directional; the activities of the inhabitants in the environment also change and create it. This reciprocal process extends to the interactions and actions of life forms vis-à-vis other life forms.

Through direct collaboration, serendipitous symbiosis, or competition; in software development ecologies, nothing exists alone.

<pre><p style="text-align: center; margin-top: 0px; margin-bottom: 4pt;">•  •  •</p></pre>

So, how is this way of looking at software engineering actually helpful? What does it provide beyond a mere and fanciful description of our practices?

Unlike any other life forms, known from any ecology, we have the ability to reason about these processes in a way that makes it possible for us to also intervene in them. We are not merely evolving by reacting to our environment. We have the ability to get involved with intent.

> ### We can change our own culture and how we interact with each other by deliberately changing the environment.

Let’s consider a practical example: Fixing a flaky unit test. This can have a profound ecological impact and drive incredible behavioral change.

Removing the flaky character of tests will make the state of the trunk (main branch) more predictable. There will be fewer false positive and negative reports from CI that undermine trust in the reported status. As a result, developers will be more prone to frequent integrations and shorter-lived feature branches. They won’t need to worry about merging in a broken main branch. This, in turn, leads to fewer bugs introduced by complex merges and more code sharing and collaboration between different teams.

A careful, deliberate intervention in the environment by an experienced steward of the ecosystem can have a profound positive impact and make the ecology thrive and flourish.
Software architect in a rainforest

![Architect in rainforest](architect.jpeg)

Another lesson that we ought to have learned from biological ecology is that interventions into an environment can have catastrophic consequences. In some cases, we have witnessed how an ignorant action has led to a full collapse of the ecosystem.

This is also a risk in software development organizations. The blunt and autocratic introduction of a new “incentives program” by a CTO that permeates individual and quantitative achievements is a dangerous thing. We have witnessed many KPI-fueled wildfires and ecological disasters in software engineering ecologies.

When a company connects salary and career advancement to how many tickets or story points a developer can complete in a given time, disaster is looming. Individual developers stop cooperating and instead start competing for the easy work. Important but difficult tasks remain unfinished. The quality of work diminishes as no one has time to do things properly or to refactor issues they discover.

In short, an intervention into the environment has great potential but must be done attentively and with great care.

<pre><p style="text-align: center; margin-top: 0px; margin-bottom: 4pt;">•  •  •</p></pre>

Changing an unfamiliar environment is especially dangerous but unfortunately very common. The old developer adage captures this experience: “The only thing worse than a manager who doesn’t understand anything about development is a manager who understands a little bit.”

An experienced manager, tech lead, or architect acts as a gardener. When starting to tend to a new garden, they apply the “One Year Rule of Gardening”: Don’t change anything before you have seen one full season. Only tend to the weeds and attentively observe what grows when and where. Pay attention to where the shadows fall in spring and what spots are sheltered from autumn winds and where water accumulates after a summer rain.

Translated to development, this amounts to observing how teams use different tools in different phases of a sprint or what communication patterns are exhibited when making a release. What are the thorny modules that no one likes tending to, and what API functions are always misunderstood by customers? Only start making small probing changes after observing a few of these cycles.

With time, you will become an expert gardener and steward of the ecology. But even then, true mastery is displayed by only rarely resorting to larger changes. The best technical leaders bring about a thriving ecology through small changes to the environment. Understanding the complex web of interactions, cause and effect, and how various life forms will react allows you to make just the right small intervention at just the right time.

<pre><p style="text-align: center; margin-top: 0px; margin-bottom: 4pt;">•  •  •</p></pre>

Let’s investigate another complex technical issue through the lens of cultural ecology. Should you organize your code as a monorepo or divide it up into several smaller independent repositories?

These two approaches represent quite different ecological environments for developers and subsequently drive different behaviors and adaptations.

When Darwin visited the Galápagos Islands in 1835, he noticed how finches had evolved differently on different islands. The isolation of the islands caused a heterogeneity of life forms. This is also true for poly-repo ecologies. They tend to exhibit variations in coding style, build systems, branching strategy, etc. Being isolated, they also exhibit favorable traits like internal cohesion, less complex dependencies, and security by segmentation.

![Paradise islands](islands.jpeg)

A monorepo, on the other hand, is a vast rainforest. Life-forms abound and can always immediately interact with each other, especially in watering holes and other nodal points (e.g., the build system). A healthy monorepo is, however, always internally structured in components and parts — it’s not a giant ball of mud. In the jungle, some life forms inhabit the treetops and others the forest floor. In the monorepo, most developer teams spend their time in a niche, a fairly limited set of components and products.

But since it’s essentially one big environment, coding style, tooling, and code sharing are usually more homogeneous. This makes it easier for life forms that inhabit one niche to move into a new one if the opportunity presents itself (a developer moving from one project to another). Ecological interactions like symbiosis and cooperation drive behaviors that permeate communication across the entire organization in the monorepo ecology. The monorepo can be a counter force to Conways’s Law.

Larger and more diverse ecosystems are generally more resilient to shocks compared to smaller ones. An aquarium ecology is more fragile than an ocean. If the code in a monorepo is already built with five different compilers because of varying project needs, the shock of introducing another one is probably manageable.

In contrast, a smaller repo that has only ever been built with one compiler will likely experience a large shock when a second one is introduced. There will be lots of new warnings, bugs, and incompatibilities discovered. In the poly-repo world, there is a different type of resilience. Each individual repository is typically less robust, but since they are disconnected, the shock waves don’t spread and the impact is localized.

<pre><p style="text-align: center; margin-top: 0px; margin-bottom: 4pt;">•  •  •</p></pre>

In closing, I hope that you will find it useful to think about software development through the lens of cultural ecology.

I hope that you will stop writing policy documents and instead start shaping the environment to drive behaviors. Next time, you might consider enforcing automatic code formatting in pre-merge CI instead of writing a code style document. If you need to write something, focus on explaining why the environment is landscaped the way it is and let the life forms loose to explore it on their own.
