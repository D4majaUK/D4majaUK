---
layout: post
title: TeamCity - Automated ProGet publish
categories:
  - teamcity-post
---

# Problem

Developer creates the library, passes to TeamCity, builds into package, publish to ProGet. How hard can that be? Well, it appears not to be as simple as you'd expect. Especially when you don't have much documentation on the tools you are using. Anyway, let's quickly move onto the solution.


# Solution

The simple process is making sure that NPM/NodeJS is installed on the build agent. Get the ProGet token. Run a series of NPM commands and Bob is your dad's brother. And here lies the problem. There is a plug-in that works with TeamCity, and all you have to do is add the local url and token and you're off to publish world. The only problem with this is that if, for examplke the token is incorrect, you won't necessarily know about it. Believe me, I spent many 10s of minutes, hours and days trying differfent combinations and ideas to get this to work and didn't succeed. The worst being that at the Publish stage, it gave a strange error that suggested it was just a permissions thing. Even though the token was given full access to do what it wanted. 

Eventually, admitting defeat on this one, I resorted to using NPM command line so that I could have a better idea of what was going on. As ProGet had an API key, I thought that would be enough. But oh no, in order for NPM to truly connect, you need to have an encoded Base64 string. So in Powershell I created the said string, and gave it another spin. 

Did it work? Nope! Back to the drawing board.

Scouring the net, cursing and losing a grip on whatever day it was, I was determined to get this sorted. So, I tried another search and came up with an article that suggested I could create the BAse64 string using the Chrome browser. WHAAAAATTTT!!!??? 
Indeed it does. If you press F12 and navigate to the Console tab, you can type: btoa($string that needs to be encoded) and you will get a Base64 encoded string. I tried that, and crossed fingers, toes and eyes. The last idea not something one should try for too long, especially not the length of this particular build. Anyway, for the first time, in a long time, I had a green TeamCity build. However, I have been there before, to then find out that the package did not update. I checked nervously, the version number (as I had asked the engineer to bump up the version). Hoorah! It was as I had wished for, a new version. Yahooooo!

And that folks, is a done deal.

If at first you don't succeed, try try again.

