# Synthetic Threat Intelligence Reports

## TLDR

I taught a Machine Learning Language model to write Infosec Threat Intelligence reports. 

If you just want to skip to the results, the "best" (as in closest to English) is here: 
https://gist.github.com/g-clef/8ea6b388a931f570615fd55b3fbbefe3

The funniest one (the other definition of "best") is here: https://twitter.com/gclef_/status/1487817838744125448


## WTF?

I'm sure you've seen the joke posts that say things like "I made an AI watch 200 hours of Friends, then asked it to
write a script. Here's what it said." Those are cute, but they're part of a really interesting trend in Machine 
Learning towards artificially generated words, music, pictures, etc. (I refuse to call it "content.") The joke is 
that the machine-generated words are hilariously *close* to readable, but wrong in funny ways. That's often true,
but it also misses the point - the fact that an ML algorithm's output is understandable *at* *all* is amazing. 

I find these generative algorithms fascinating, and wanted to try my hand at them. Figuring my best first target would
be a field I'm already in, I decided to try to generate artificial Threat Intelligence reports. They seemed like the 
perfect target: a somewhat specialized jargon, so it would be really obvious if the ML model had actually learned to 
speak the lingo, and a fairly large corpus of English-language samples to train off of. I already had an ML setup at 
home (a gaming rig with an RTX2070 Super), and could get the TI samples (more on this below), so this seemed like an 
achieveable goal.

In theory, the only thing I needed to do was collect a bunch of Threat Intelligence reports, feed them to a 
pre-trained Machine Learning Language model and say "go".

Of course, it wasn't that easy.


## Where'd the data come from?

Before doing any machine learning, you need data for the algorithm to learn from. In this case, I wanted lots of 
Threat Intelligence reports. I collected the sample Threat Intelligence reports from a pair of Github projects: 
 * [APTNotes](https://github.com/aptnotes/data)
 * [APTCyberMonitor](https://github.com/CyberMonitor/APT_CyberCriminal_Campagin_Collections)
 * (I was originally also pulling a collection from FDiskYou, but they appear to have shut down)

These are both updated fairly often, and both store their results in a common format (pdf), which makes later parsing
of them easier, since they can be parsed and prepared with the same code. 

To collect them consistently, I built a kubernetes job that's designed to run nightly to grab anything new. If  you'd 
like to run it yourself, it's here: https://github.com/g-clef/ThreatIntelCollector . It's built assuming that it runs
in a [malwarETL](/projects/malwarETL.md) k8s cluster, since that's the cluster I have, so it has some 
idiosyncracies relevant to my setup.

Given those two data source, there is a obvious bias in the dataset right off the top: If those two projects don't 
pull in a report, my dataset won't have it, which means the ML model can't learn it. I'm sure there's TI 
reporting that they're not grabbing (it's a huge field, with lots of startup companies putting out reports to prove they
know what they're doing), so that does put some Machine Learning limits on the use of this data right away - this 
dataset probably isn't a complete view of the TI report landscape, so it's not safe to use this data to make 
industry-wide conclusions about TI reports. For the use-case of teaching an ML model to ape reporting, though, that's 
less of an impediment. It does mean that the eventual model may miss some jargon or examples that it would otherwise 
get, but for my purposes that's acceptable.


### The gory details

