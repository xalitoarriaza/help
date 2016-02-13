fortrabbit help pages
=====================

Welcome to the source of the official fortrabbit documentation. These files here are written in MarkDown, with a little frontmatter meta-data on top. We include this repo as a Git subtree and publish it on: http://help.fortrabbit.com


Contributing
------------

Found a typo or an error? Do you want to add something about your framework or service? You are more than welcome to contribute here. 



Writing guidelines
------------------

Please check out other articles to find about the patterns to structure documents.



Front Matter syntax
------------------

Each file has yml block on top. See here which tags are available and how to use them.



```yaml
# which template to use - "article", if in doubt
template: article

# do not (yet) list this article in the navigation - eg work-in-progress
dontList: true

# disable TOC generation for this article
noToc: true

# title shown in article header on display
title: What is an App anyways?

# title shown in navigation -> short but descriptive
naviTitle: About Apps

# lead text for article detail view -> what to expect from content
lead: Forget servers. Think services instead. Learn the basic fortrabbit concepts.

# names (=filename without .md) of linked articles related to current
seeAlsoLinks:
    - app
    - composer

# additional keywords for document search to help users find this article
keywords:
    - foo
    - bar

# tag article to filter & group
tags: 
    - beginner
    - cms
```