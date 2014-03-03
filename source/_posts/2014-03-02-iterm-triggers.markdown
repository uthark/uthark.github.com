---
layout: post
title: "Using iTerm triggers"
date: 2014-03-02 23:51
comments: true
categories: 
    - iterm
    - osx
    - tips
---

I want to share one of the cool features which [iTerm.app][iterm] has - triggers.
The idea behind triggers is simple - it allows to perform arbitrary action based on the output in console window. For example, it can colorize specified content (based on [regular expressions][re]).

Start `iTerm`, open settings (`⌘+,`), go to `Profiles`, then tab `Advanced`.
Click on `Edit` button in `Triggers` section.

{% img https://lh4.googleusercontent.com/-n8Ynka3iux0/UxQQDk6VewI/AAAAAAAAPgg/FoQAq7y2a78/w2088-h1242-no/Screen+Shot+2014-03-02+at+11.48.02+PM.png iTerm Settings %}

Then you can define your own triggers:

{% img https://lh5.googleusercontent.com/-zXeqBjdPMlE/UxQQMQ4EyXI/AAAAAAAAPhg/0tS9qMw5atw/w2086-h932-no/Screen+Shot+2014-03-02+at+11.48.15+PM.png Triggers Configuration %}

Example output:

{% img https://lh6.googleusercontent.com/-hiVCoX2YS60/UxQSnH64xwI/AAAAAAAAPxw/KIlW7dpv-tM/w1816-h1386-no/Screen+Shot+2014-03-03+at+12.26.03+AM.png Example output %}

Happy coding!

[re]: http://en.wikipedia.org/wiki/Regular_expression
[iterm]:http://www.iterm2.com/
