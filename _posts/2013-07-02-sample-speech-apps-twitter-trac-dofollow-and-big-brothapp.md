---
layout: post
title: 'Sample Speech Apps: Twitter, Trac, Do&Follow ... and Big Brothapp!'
date: 2013-07-02 17:20:31.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Apps
- Speech
tags: []
meta:
  _edit_last: '43713691'
  draftfeedback_requests: a:1:{s:32:"habla-computing@googlegroups.com";a:3:{s:3:"key";s:13:"51d2b729d06da";s:4:"time";s:10:"1372763945";s:7:"user_id";s:8:"43713691";}}
  draft_feedback: |-
    a:1:{s:32:"habla-computing@googlegroups.com";a:3:{i:0;a:2:{s:4:"time";s:10:"1372764547";s:7:"content";s:68:" JSantos: ¿Es DO&FOLLOW UP? ¿O DO&FOLLOW?
    El resto está perfecto.";}i:1;a:2:{s:4:"time";s:10:"1372764903";s:7:"content";s:50:"* Do and follow: typo: to asssign tasks to people
    ";}i:2;a:2:{s:4:"time";s:10:"1372766499";s:7:"content";s:126:"The last sentence in trac description has two "about".
    I would like a joke as we had been loved to be twitter infraestructure
    ";}}}
  _publicize_pending: '1'
  _oembed_2d44823bfeb81056a186b880b93f0dcb: "{{unknown}}"
  _oembed_60354aeb28acd368eaf61941f11ce6f7: "{{unknown}}"
  _oembed_2ef86f6b2d068afc0fc4d94d6faac3c8: "{{unknown}}"
  _oembed_2d08cac1966d521849243ac78aa94bcd: "{{unknown}}"
  _oembed_14a0acd118d4f4fa9f9e1c43ab7650fa: "{{unknown}}"
  _oembed_f6d3703f42429b1e6b3d66b3500cb333: "{{unknown}}"
  _oembed_0cb6d591a74df64d83666da38aa6172d: "{{unknown}}"
  _oembed_59e0e18fe559188812942694ed85425c: "{{unknown}}"
  _oembed_9c64babb70cbb8a15e665bc77b8213fe: "{{unknown}}"
  _oembed_d26e800a86bd81014dbf2a26b396e912: "{{unknown}}"
  _oembed_eb55b1714ca9afcee6633fff230ec65d: "{{unknown}}"
  _oembed_1983f43518bdebb670f2c7ad36dedea5: "{{unknown}}"
  _oembed_8deb8bb99a6b63fac4947783df208da9: "{{unknown}}"
  _oembed_78dfc533a07622d6bdd0df89e63fdd6f: "{{unknown}}"
  _oembed_67561a2966279df9a78475feb7c7392a: "{{unknown}}"
  _oembed_b8a612459e0a442f53272b241ec44697: "{{unknown}}"
  _oembed_25248a6eba876b08636452b5df08b15d: "{{unknown}}"
author: Juan Manuel Serrano
permalink: "/2013/07/02/sample-speech-apps-twitter-trac-dofollow-and-big-brothapp/"
---
Which kinds of applications are most suitable for Speech? How do we design a Speech app? How do we implement a Speech design using its Scala embedding? To answer these questions in a pleasant and entertaining way, we completely re-designed our web page and chose four applications in quite different domains for illustrating the virtues of our DSL. The whole Habla Computing <a href="http://speechlang.org/company/en/main.html?section=people&part=staff">team</a> has been involved in the design and implementation of these apps. Here they are!

<img alt="" src="{{ site.baseurl }}/assets/2013/07/twitter.png" width="100" height="100" />

We all know *<a href="http://twitter.com" target="_blank">Twitter</a>.* It's a micro-blogging platform which allows people to publish short-message updates. Can we program the structure of Twitter interactions and its communication rules in Speech? Of course! Accounts, tweeters, tweets, statistics, and so forth, can be very easily mapped into social interactions, agent roles, speech acts and information resources  - the different kinds of social abstractions put forward by Speech. "Following" rules for private accounts or blocking users can also be expressed very easily using empowerment and permission rules. Monitoring rules can also represent communication rules (e.g. forwarding tweets to your followers) and notification preferences (e.g. being informed that you've got a new follower) straightforwardly.

<img alt="" src="{{ site.baseurl }}/assets/2013/07/dofollow.png" width="100" height="100" />

<a href="https://github.com/hablapps/app-dofollow">*Do&follow up*</a> is a task-based management application which allows project responsibles to assign tasks to people, monitor progress, attach information resources to each ongoing task, launch automatically new tasks according to a pre-established workflow, etc. It belongs to a completely different domain from Twitter ... isn't it? Not really. Indeed, both workflow management software and social networks ultimately deal with people and their needs for communication and collaboration. The differences between them simply lie in the way communications are structured and governed. Thus, we can model tasks and their responsible people as particular types of social interactions and agent roles, respectively, and represent workflows through life-cycle initiation and finalisation rules.

<img alt="" src="{{ site.baseurl }}/assets/2013/07/trac.png" width="100" height="100" />

The <a href="http://trac.edgewall.org/" target="_blank">*Trac*</a> application is an issue tracking system for software development projects. It's somewhat similar to the do&follow up application, but you won't find a pre-established workflow here. The trac application is interesting from the point of view of Speech because it allows us to illustrate its ability to support both structured and unstructured communication modalities: on the one hand, issues go through a number of well-defined stages and follow specific rules for assignments and completion; on the other, the owner, reporter and arbitrary members of the application can engage in free conversation concerning an issue. Again, it's all about about speech acts with different normative weights and constraints.

<img alt="" src="{{ site.baseurl }}/assets/2013/07/bigBrothapp100x100.png" width="100" height="100" />

The <a href="https://github.com/hablapps/app-bigbrothapp">Big Brothapp</a> prototype was designed after the rules of the <a href="https://en.wikipedia.org/wiki/Big_Brother_%28TV_series%29" target="_blank">big brother</a> reality game show. The purpose of the application is to assist the producers in the management of the TV show, particularly by enabling a computational representation of the house, the eviction processes and any other aspect which belongs to the "rules fo the game". These rules can be accommodated in a very convenient way by Speech, so that that the structure and dynamics of the contest can be formalised with empowerment and life-cycle rules, standard speech acts, etc., very easily!

### *Watch the design and coding video guides!*
We used Big Brothapp to illustrate the Speech <a href="http://speechlang.org/documentation.php?s=Design">design</a> and <a href="http://speechlang.org/documentation.php?s=Coding">coding</a> processes through a series of videos that you can find in our web page. The <a href="http://speechlang.org/apps.php">apps</a> section also shows videos for the other applications, as well as their source code on github. If you have any comments, questions or wonder how you preferred app could be modeled in Speech, please tell us!

Last, you will also find in the <a href="http://speechlang.org/apps.php">apps</a> section direct links to runtime deployments of these apps through their respective Consoles - but this will be the subject for the next post. Enjoy!

