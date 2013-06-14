---
layout: post
title: "How to create an OS X Console App with RubyMotion"
date: 2013-06-10 22:52
comments: true
categories:
lang: en
published: true
---

Frankly speaking, the approach I am talking about here is to create a Mac console-like application as we are not able to build a real Mac console application using RubyMotion when I post this article.

It takes only 3 steps, let's get started.

Firstly, create a RubyMotion project using OS X template. In my example, I call it **ConsoleMotion**:

``` bash
$ motion create --template=osx ConsoleMotion
```

Secondly, edit the **Rakefile** and add the following lines of code:

``` ruby
# -*- coding: utf-8 -*-
$:.unshift("/Library/RubyMotion/lib")
require 'motion/project/template/osx'

Motion::Project::App.setup do |app|
  # Use `rake config' to see complete project settings.
  app.name = 'ConsoleMotion'
  app.info_plist['LSUIElement'] = true # Add this line
end
```

When the `LSUIElement` property of your application's info plist is set to `true`, your app will not appear in the Dock as the [documentation](http://developer.apple.com/library/ios/#documentation/general/Reference/InfoPlistKeyReference/Articles/LaunchServicesKeys.html) says.

Thirdly, edit the **app_delegate.rb** and implement `applicationDidFinishLaunching` method as follows:

``` ruby
class AppDelegate
  def applicationDidFinishLaunching(notification)
    args = NSProcessInfo.processInfo.arguments # get the arguments passing to your console-like app
    str = args[1] || "hello world"

    # do whatever you want here
    p str

    exit # exit the app when you finish your job
  end
end
```

the key point is the `exit` function which quits your app when jobs finish like what the normal console apps do.

Next, build your app and run the executable:

``` bash
$ rake
     Build ./build/MacOSX-10.8-Development
   Compile ./app/app_delegate.rb
   Compile ./app/menu.rb
    Create ./build/MacOSX-10.8-Development/ConsoleMotion.app/Contents
    Create ./build/MacOSX-10.8-Development/ConsoleMotion.app/Contents/MacOS
      Link ./build/MacOSX-10.8-Development/ConsoleMotion.app/Contents/MacOS/ConsoleMotion
    Create ./build/MacOSX-10.8-Development/ConsoleMotion.app/Contents/Info.plist
    Create ./build/MacOSX-10.8-Development/ConsoleMotion.app/Contents/PkgInfo
      Copy ./resources/Credits.rtf
    Create ./build/MacOSX-10.8-Development/ConsoleMotion.dSYM
       Run ./build/MacOSX-10.8-Development/ConsoleMotion.app/Contents/MacOS/ConsoleMotion
"hello world"
$ ./build/MacOSX-10.8-Development/ConsoleMotion.app/Contents/MacOS/ConsoleMotion "we're done"
"we're done"
```

That's it.
