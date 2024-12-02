# Bloglog

This post is created to register the special configurations this blog have had.

# Injecting Analytics script:
This is a script that should be present in the head of every page, so I could have visibility of the site performance.
Looking in the documentation, they asked to look for the component head.html inside the _includes floder. But, in the source code, there is no _includes folder. Then after prompting a lot, I was able to learn this command: `bundle show jekyll-theme-chirpy` with that, I found that the default values for files in _includes, _layouts and others, are there, inside the installed get (folder) i.e `d:/pkg/gem/gems/jekyll-theme-chirpy-6.5.5`. There, it was the _includes folder with the head.html among others. I could have changed the head file there including the reference to google analytics, but as it is not in the folder of the source code, it won't be able to be commited.