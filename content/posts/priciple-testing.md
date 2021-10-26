---
title: "Principle of Software Testing"
date: 2021-10-25T00:11:23+08:00
draft: false
tags:
    - testing
keywords:
    - testing
---

## Defination of expected results is a must of test case.
Test cases must describe the input of the program and an expected output of the program.

## Programmers should avoid to test the programes they wrote.
Most of the programmers cannot see the program in another perspective after they writing it.

## The development orgnization should avoid to test their own program.
If the orgnization have a target to deliver program in a time and quota limitation process, then it will intend to see less problem.

## The results of testing should be checked very carefully.
It's obvious but commonly ignored principle.

## Test case should not only consider the valid input, but also consider the invalid input to try to break the program.
Most of the time, an invalid input will more likely to trigger some issues.

## Checking the program do what we intend to do is only half of the testing, the other half belongs to check if the program do something we're not intend to do.
In case any side-effect.

## Test cases should be reusable.
This means we should do as many as automation we could, or at least document it.

## We should not assume there's no problem when we plan the testing.
Test is a process to discover problem, not a process to prove the correctness.

## The part has the most problems will more likely to have more problems.
Even thought we cannot explain this, but the truth is if one part of the program has the most bugs, then this part will more likely to have more bugs.