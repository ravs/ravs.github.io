---
layout: post
title:  "To TODO, or not to TODO"
date:   2019-05-04
herounit: PostAlleyGraffiti
---

Last week I was working with an away team, and in one of my code review, I got a comment from one of the senior engineer that, we should add ```TODO``` for something which I will be adding in next reviews. My response to that was, "*I am not a big fan of adding TODOs in code since they don't add much value, especially when I'll be working on that code in next few days iteratively and complete the full implementation*". That led to a healthy debate over whether TODOs add value to our code or not.

First off let me starting with my opinion and why I think it does not add value.
- **Hides Work**: TODOs are for something which is missing in the code, which raises questions such as:
  - What made us ship the code which has missing something and why we can't work on it before pushing this change?
  - When are we going to work on this miss?
- **Lacks Traction**: Since TODOs are hidden in code, there is no way to track the missing work.

The argument made by the senior engineer was that, *it's easier to search for TODOs in code and all IDEs support easy look ups*.
This is a fair point, but I would question when and what motivates one to look up for TODOs. If you stumble upon one then that's a different case, but is it an acceptable trend to periodically search your code base for TODOs?

I further did more research on internet and found mixed feelings about the use of TODOs. Some like it and some don't. There is also a tool which creates a tracking task in your issue management system and resolves them once the TODO has been deleted. Seems intelligent. I would love to use something like that in future, until then, I am advocating not to TODO.
