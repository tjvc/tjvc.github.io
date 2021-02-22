---
layout: post
title:  "When to reload ActiveRecord objects in tests"
date:   2021-01-02
---

A common gotcha in Rails functional testing is asserting against an ActiveRecord object mutated via another instance.
This can easily lead to both false positive and false negative test results.

To ensure accurate results when asserting against an ActiveRecord object, first call `.reload` if the record has been updated via another instance in the course of the test. This includes associations, except where these have not previously been loaded from the instance under test.
