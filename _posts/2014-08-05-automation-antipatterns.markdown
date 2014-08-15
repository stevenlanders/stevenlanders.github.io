---
layout: post
title:  "QA Antipattern: Robotic Arms"
date:   2014-08-05 00:00:00
categories: ['automation','qa']
---

Manual testing, in the clicking-monkey sense, should disappear. There should not be a role for a professional validator who clicks through a series of steps verifying functionality is not broken.  

These manual testers should target a different role.  This role could be a product evaluator or a testing coverage analyst. I believe there *ARE* elements to user experience and user acceptance testing that can be productively manual.  That said - these should not be defect-detecting exercises, but rather value-evaluating exercises.

In short: *Humans should do human things. Robots should do robot things.*

In a noble effort to transition away from costly manual QA, some organizations attempt what I'm referring to as the "Robotic Arms" antipattern.  This post is an attempt to describe this antipattern.

### Problem to be solved

You have 100 manual testers without automation skills, but with product knowledge gleaned from previous iterations.  You want one of two things:  fewer testers, or higher quality with existing testers.  

### Antipattern description

1. Have a team of skilled automating consultants write a library of automated testing scripts
2. Give those scripts to manual testers to execute
3. Do not train your manual QA beyond the ability to execute the test 
4. Say farewell to automating consultants 

### Expected yields:
- No need to train manual testers ($$$ savings $$$)
- Executions will increase speed of each execution
- Duration of QA cycle will decrease
- Profit

### Actual yields:
- Testers run scripts, some % of tests break for reasons like bad data or brittleness
- Executions that pass are not evaluated for coverage/value, as no one can understand the scripts
- As application changes, no one prioritizes updating the testing scripts
- Over time, fewer and fewer scripts execute, and manual executions resume

It seems plausible that by having someone else write the test, you can still have these manual testers execute the test to get the benefits of automation (faster, more consistent)  After all, training your manual QA personnel in writing automated tests is too costly - so you want to avoid it right?

### These intuitions (reasonably) lead to this:
- An automated test will execute a test more quickly than a manual tester can *true*
- Toyota can build cars more quickly and reliably with robots *true*
- Faster QA executions mean that QA durations are shorter. *Yes...but now you can run it more often!*
- Manual testers are not technical enough to write automated tests *What? What is it to be 'technical'?  Learn new things!*
- Automated tests can be reduced to a single button that anyone can push 

### Mental shift required

Faster QA executions is a good thing, but not good because of a lower overall QA duration.  Faster QA executions lets us run it MORE FREQUENTLY and get QUICKER FEEDBACK. 

### Recommended Alternative

Given 100 manual testers, you are not going to make them automation experts overnight.  You must invest constantly in automation and the evolution of the organization.    

Stop buying automated tests, and invest in automation capabilities.  