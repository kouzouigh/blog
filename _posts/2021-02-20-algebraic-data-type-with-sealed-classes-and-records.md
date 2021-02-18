---
layout: post
title: Algebraic Data Type with sealed classes and Records
date: 2021-02-20 00:00 +0200
category: programming
tags: Tech
---

### Motivation
**Java 15** introduces _Sealed_ Classes and *_Records_* as  preview features, and these 2 features will allow us to:
* Create closed hierarchy
* Support Pattern matching
* Modeling Data driven objects (Immutable with equals and hashcode automatically generated)

The combination of **_sealed classes_** and **_records_** is like _**trait**_ and _**case classes**_ in Scala, and this combination is referred as algebraic data types. This is a revolution for Java language and it will impact how we will model our domains objects.   

