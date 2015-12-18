---
layout: post
title: "2 gists to help you github better."
date: 2015-12-18 20:43:56 +0000
comments: true
categories: 
---

I recently stumbled upon a few code snippets for streamlining my github activities, expecially with Ruby and Jekyll

This one is helpfull for deploying a Jekyll or even a Ruby project to GitHub pages. It adds all files, commits with a timestamp, integrates, generates the blog files, and pushes it up to github:

{% gist f64123058be1f84a3379 %}

This following one is helpfull for a quick automated pus hto github. I like t ocall these courtosy flushes:

{% gist 430fc8db653de052a6cb %}

You can change the commands to whatever you want but as is, run the first by typing "rake blog" and the second by "rake push".

If you add these to the bottom of the Rakefile in your project they should work out of the box.

-R

The above linked to my github gists and the below is inline markdown. I really jsut did htis to learn how for future reasons.

Alternate code listed:

```ruby
# append this to your rakefile and simply type "rake blog" for full deployment process
desc "Generate website, add, commit and deploy"
task :blog do
    system "git add ."
    message = "Site updated at #{Time.now.utc}"
    system "git commit -am \"#{message}\""
    Rake::Task[:integrate].execute
    Rake::Task[:generate].execute
    system "git push origin source"
    Rake::Task[:deploy].execute
end
```

and

```ruby
# append this to your rakefile to add autopushing
desc "pushes to git with commit"
task :push do
    system "git add ."
    message = "Site updated at #{Time.now.utc}"
    system "git commit -am \"#{message}\""
    system "git push origin source" # or "git push origin master"
end
```