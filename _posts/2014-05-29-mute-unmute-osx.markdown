---
layout: post
title: "Global audio input mute/unmute keyboard shortcut for OSX"
categories: [skype, hangouts, osx]
---

Launch Automator and create a new *Service*, include a *Run AppleScript* action
configured to receive "no input" in "any application".

Use the following code for the script:

{% highlight applescript %}
on run {input, parameters}
    
    set inputVolume to input volume of (get volume settings)
    if inputVolume = 0 then
        set inputVolume to 100
        display notification "🔊🔊🔊🔊🔊🔊🔊🔊🔊🔊" with title "🔊 UNMUTED!" sound name "Tink"
    else
        set inputVolume to 0
        display notification "🔇🔇🔇🔇🔇🔇🔇🔇🔇🔇" with title "🔇 MUTED!" sound name "Pop"
    end if
    set volume input volume inputVolume
    
    return input
end run
{% endhighlight %}

> **NOTE**: Since we use the *display notification* API it requires at least OSX Maverick
to run. For older versions you can replace that call with something like Growl.

We can now save the automator service and set the name "MuteUnmute" to it for instance.

Now launch the *System Preferences* and enter *Keyboard* panel, on the *Shortcuts* tab
select "App Shortcuts" on the left and click on the <kbd>+</kbd> icon to create a new one with 
the following details:

 - *Application*: All Applications
 - *Menu Title*: MuteUnmute
 - *Keyboard Shortcut*: <kbd>⇧⌘M</kbd> (or whatever you want)

Close the *System Preferences* for the shortcut to take effect. Now you can easily
mute and unmute your microphone input for any program by using a global keyboard
shortcut.

