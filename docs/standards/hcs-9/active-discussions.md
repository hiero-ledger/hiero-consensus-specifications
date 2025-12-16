---
title: Active Discussions
sidebar_position: 5
---

#### [ hcs-9 - Poll Metadata Schema ]


### Support for rich-text

Should this standard support rich text features, such as bold, italics, colours, links, etc?

If so, which features should be supported and how should we define it such that it is as simple to implement as possible?


### Should 'schema' be optional (defaults to "hcs-9")?

For the majority of platforms starting out, they likely will mostly be using default implementations of the standard. Therefore, explicitly defining 'schema' in all modules could be seen as redundant. 

Currently schema is required, but it is reasonable that it be made optional and if missing, assumed to be "hcs-9".

The downside of this is developers may shortcut the schema validation check entirely, and if developers start implementing new schema modules, the apps may not be written to account for it and not be able to process those schemas. If there were an explicit check for a schema definition from the start then platforms could fail elegantly.

Discussion encouraged.

### Support for additional metadata

It could be interesting to support additional fields such as poll banners, images, diagrams and other items that enrich the display of the poll. Links to creator websites, discussion forums and social media also are important pieces of information that could be defined within the poll and allow that information to follow the poll.

The caveat here is that the number of options for metadata is limitless, so how do we create a standard that allows for information to be stored, while remaining lightweight and flexible? In addition, this additional metadata is valuable, but not critical to the functionality of the poll, so defining it specifically within this standard may take away from the focus.

Discussion on this topic is open and welcome.
