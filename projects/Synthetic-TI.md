# Synthetic Threat Intelligence Reports

I taught two Machine Learning Language models to write Infosec Threat Intelligence reports. It...worked, I guess - the
final result from GPT2 was surprisingly understandable, but still not good enough to actually call a real "report". It
was still recognizable English, which I think is super-interesting. On the way there, though, I had some entertaining
failures.

## Intro

### TL;DR

If you just want to skip to the results, the "best" (as in closest to a real report) is here: 
https://gist.github.com/g-clef/8ea6b388a931f570615fd55b3fbbefe3

The funniest one (the other definition of "best") is here: https://twitter.com/gclef_/status/1487817838744125448

The code that does all this is here: https://github.com/g-clef/synthetic-ti-reports

### Why would you want to do this?

There's a part of Machine Learning that's using ML to generate text, music, pictures, etc. (I refuse to call it 
"content.") This machine-generated text is sometimes hilariously *close* to real text, but wrong in funny ways. 
The thing is, I think joking about it misses the point - the fact that an ML algorithm's output is understandable 
*at* *all* is amazing. 

I find these generative algorithms fascinating, and wanted to try my hand at them. Figuring my best first target would
be a field I'm already in, I decided to try to generate artificial Threat Intelligence (TI) reports. They seemed like the 
perfect target: a somewhat specialized jargon, so it would be really obvious if the ML model had actually learned to 
speak the lingo, and a fairly large corpus of English-language samples to train off of. I already had an ML setup at 
home (more on this below), and could get the TI samples (more on this below, too), so this seemed like an achieveable 
goal.

In theory, the only thing I needed to do was collect a bunch of TI reports, feed them to a pre-trained Machine Learning 
Language model and click "go".


### Where'd the data come from?

Before doing any machine learning, you need data for the algorithm to learn from. In this case, I wanted lots of 
TI reports. Luckily, a pair of Github projects are collecting them already: 
 * [APTNotes](https://github.com/aptnotes/data)
 * [APTCyberMonitor](https://github.com/CyberMonitor/APT_CyberCriminal_Campagin_Collections)
 * (I was originally also pulling a collection from FDiskYou, but they appear to have shut down)

These are both updated fairly often, and both store their results in a common format (pdf), which makes later parsing
of them easier, since the reports collected from both of them can be parsed and prepared with the same code. 

To collect them consistently, I built a kubernetes job that's designed to run nightly to grab anything new. If you'd 
like to run it yourself, it's here: https://github.com/g-clef/ThreatIntelCollector . That collector is built assuming 
that it runs  in a [malwarETL](/projects/malwarETL.md) k8s cluster, since that's the cluster I have, so it has some 
idiosyncracies relevant to my setup.

Before going further, I should talk about bias. There is a potential bias in this dataset right off the top: If those 
two projects don't pull in a report, my dataset won't have it, which means the ML model can't learn from it. While I 
confess I haven't checked, the TI field has lots of startup companies putting out reports to prove they know what 
they're doing, so I would not be at all surprised if these projects missed some. Assuming that's the case, that means 
this dataset probably isn't a complete view of the TI report landscape, so it's not safe to use this data to make 
industry-wide conclusions about TI reports. For the use-case of teaching an ML model to ape human reporting, though, 
that's less of an impediment. It does mean that the eventual model may miss some jargon or examples that it would 
otherwise get, and may bias towards some vendors, especially if those vendors are over-represented in the dataset,
but for my purposes that's acceptable.


### What did I do this all on?

None of this took a lot of firepower. The hardware I ran this on is my old (like 10-year-old) gaming rig, with 12 GB
of RAM, but I updated the video card to an NVIDIA RTX2070Super. While those are hard to find at the moment (f crypto 
mining), when I got it they were solidly middle of the road, running about $600.

To install FastAI and the like on that rig, I put this all in a python virtual environment & did:
```
python3 -m venv .venv
source .venv/bin/activate
pip3 install torch==1.10.1+cu113 torchvision==0.11.2+cu113 torchaudio==0.10.1+cu113 -f https://download.pytorch.org/whl/cu113/torch_stable.html
pip3 install fastai
pip3 install transformers
pip3 install jupyterlab
```
So if I ran `jupyter lab` while the venv was active, I'd get a full FastAI & transformers development environment. 

### Why would this even have a hope in hell of working?

The core idea for this all is the idea of fine-tuning a pre-trained model. The assumption behind it is that the English
language is somewhat generalizable, and so you can transfer the basics of language from one model to another. In other 
words, I should be able to take a machine learning model that someone else has spent a lot of time training on a large 
corpus of generalized English texts, like Wikipedia or movie scripts, etc, and only need to run a small number of 
training passes over my specialty text to teach it the specifics of the particular corpus, since it should already 
have the basics of how English works from the pre-training. 


## The Gory Details

### Preparing the data

While it would be nice to be able to hand a PDF to an ML model, right now the language models can't read PDFs. So I 
had to pull the text of the reports out of the PDFs into pure text. I could do this on the fly, but that would
be hideously slow, since it would make the GPU wait for every decoding before doing any work, which is a bad idea. 
Ideally you want the GPU to be getting data quickly so that you can max out its parallel processing capabilities. 

I tried a number of Python PDF parsers, but settled on `pdfplumber`. In pdfplumber I intentionally asked it to retain 
layout, since I was hopeful that the eventual model would reproduce that layout. For various reasons that turned out 
not to be the case, but retaining layout still led to the overall best output. 

To run the preparation/text extraction, I created a Prefect job to do the parsing (so that I wasn't having to 
babysit a long parsing task). Those jobs ran in my malwarETL cluster's 
[Prefect](https://github.com/g-clef/malwarETL-prefect) setup. I ended up running several versions of this Prefect job
to remove more and more things like code snippets, multi-line whitespaces, and URL-like strings, in an attempt to 
get to just the text of the report. That seemed to matter less for some models than others.

The actual code to run in Prefect is in the [Synthetic-ti-reports](https://github.com/g-clef/synthetic-ti-reports) github
repo, under the "prefect" directory. 

## Attempt #1 - LSTM

Before going into the fancy models that make headlines, I decided to start with a language model called an
[LSTM](https://en.wikipedia.org/wiki/Long_short-term_memory), which stands for Long Short-Term Memory. 
According to the wikipedia, LSTMs have been used in Google Voice for speech recognition, and Facebook for automatic
translations. I'm using a pre-trained LSTM that's distributed with [FastAI](https://www.fast.ai/).
(The fact that it's already part of FastAI so that I didn't have to do anything else was why this one's first: it was
easiest to start with.)

In the interest of "let's see what happens", I first ran the LSTM against a fairly unprocessed input: just the text,
with positional newlines left in, and no other filtering of the text. The result was...
[not great](https://gist.github.com/g-clef/c038c0c7ef75a45ccd28e8ea317d7bdb). It also took a *long* time to train - each
epoch of training (I did 10 passes over the whole dataset) took a bit over 20 minutes. So the full training run took a
few hours.

Looking at the results, it looks like it got very confused by the code samples that were included in some of the TI
reports. That shouldn't be totally surprising: source code is basically another language, with its own grammar and syntax,
so reports that had code snippets were dumping a bunch of non-English-language behavior into the mix, which almost
certainly confused an English-language model. 

So, for the second attempt with the LSTM, I removed the code snippets as best as I could. Looking at the reports, it
seemed like code snippets started with a line that had one of a few limited words (`def`, `import`, `#define`, etc),
and the *end* of a code snipped tended to have two newlines instead of one (probably a side-effect of the layout
preservation in pdfplumber). So I added a filter for those conditions to the Prefect job that created the dataset, 
re-ran the Prefect job to create a new set of training data, and re-ran the training for the LSTM. That gave me:

```Xxunk Xxunk - a Targeted Intrusion with Network Uncovers Mirrorthief Group Targets Leveraging 

▁ VPN Domains 

▁ Meet 주소 , Tiered Architecture and foreign - policy Operations at Over 

▁ The Atlantic Treaty Organization 

▁ March 11 , 2018 at 10:21:44 PM 

▁ Vulnerability in Internet Explorer 10 

▁ 5638 - at - xpl0it - analysis 

▁ t+m - yoroi - blog 
▁ By Supplying 82053737 , a augustus 11.0 
▁ producer 

▁ October 2 , 2021 

▁ Evasive Tools of muddywater Unknown Unknown Threat Actor 

▁ Early 02 , 2020 






▁ Witnessed in the first wave of Ill - dated March 27 - 2020 attacks by espionage 
▁ in the Palestinian Authority . 


▁ Introduction 


▁ The Middle East contains an increase in cyber 
▁ capabilities , distributed denial - of - service ( dos attacks and the use of malicious 
▁ Ipsum Denial - of - service , Keybase Malware , spoofed emails and proper infection chain . 


▁ Copies of the malicious files ( jan . 13 , 2021 ) , ILLUSTRATING 

▁ darkhalo , Tortoiseshell ’s Playbook , are made on the basis of a seemingly ‘ new _ 
▁ chi - si ’ installer which is written in Python by runtime keys . 
▁ Our research into Impacting the Identity of the People ’s Republic of China and the Remcos 

▁ Orcus rat is an evolution of the scope of its campaigns , with interesting features to 

▁ evade detection and identity protection . 

▁ 2 . UNMATCHED PERSISTENCE MECHANISMS 
```

This was better in a few ways: 
  1) it didn't tend to jump into code-like things at random points in the generation, and 
  2) its sentences (where there were sentences) were closer to intelligible English. However, it still wasn't *good*
in any readable way. It had started to get the lingo, but overall the generated text was just not there. 

Out of curiosity, I tried letting it train for several more passes over the training data set (multiple more "epochs"), 
to see if the accuracy would improve with more training. It did not.

I stopped working with the LSTM here, because I felt like I'd proven the point: it's possible to do this, on my home 
setup, and the simple results were...okay. I wasn't expecting perfection, didn't get it, and that's okay. 

## Attempt #2 - GPT-2

Having now proven I can do this, and built up a reasonable pipeline to generate the data, I decided it was time to 
try one of the headline-grabbing models, to see what I could get it to do. Getting it to work turned out to be a much
bigger job than the LSTM.

In this case, I wanted to try [GPT2](https://en.wikipedia.org/wiki/GPT-2), since I've heard a lot about its ability to
create very accurate generated text. The Generative Pre-Trained Transformer (GPT2) is a machine learning model with 
very large number of parameters (more than a billion), which was trained on thousands of unpublished novels to give it
a base level of accuracy on the English language.

At first, I mostly followed the [tutorial on using GPT2 from FastAI](https://docs.fast.ai/tutorial.transformers.html), 
which was mostly fine, though I did have to modify a few things right from the start: 
  1) The tutorial is expecting you to give it a csv with each line being a new document with text to learn. In my case, 
each "document" was a separate file, so I had to read them all into RAM, put them in a python List, and turn that list
of files into a pandas dataframe.
  2) The tutorial split the training set from the validation set simply by position in the list of files - the last 10%
of the files were considered the validation set. That held the real possibility of introducing a bias in this case, 
since it's entirely possible that the file names of the reports would be grouped by vendor or by threat actor. It's 
entirely reasonable for the reports to have names like "Cisco - Threat actor 1 report.pdf", which would group all the 
Cisco ones together near the beginning so no Cisco reports would be in the validation set, or like "Wekby - Cisco 
analysis", which would group all the vendors together near the end, so no Wekby reports would be in the training set. 
That kind of imbalance in either the training or validation set is a problem, so I instead did a random 
test_train_split, on the data, to get the 90/10 split they tutorial asked for and then re-concatenated them together 
to get back to what the tutorial was looking for. 
  3) The tutorial mentions in passing that "Another way to gather the data is to preprocess the texts once" and then 
"only use the transform to decode the tensors to texts". Reading this casually might give you an idea that it's a minor,
somewhat optional step. It is not. My setup performed *abysmally* if I left this out. The reason being that without
this step, the process was re-tokenizing each text every time it was passed to the GPU for processing. This meant that
each training pass took *forever* (the validate() method call took over an hour to run, with the GPU barelly used
during that time). So I always did the pre-tokenize step.

Anyway, after running all that, I tried generating some text, specifically 100 tokens and got...a page of spaces, but
with the "end-of-text" special token right at the beginning of the message. (so it was effectively a blank response.)

This was somewhat anticlimactic. 

I tried a few things to fix this, none of which worked completely:
  1) Did gpt2 want special `startoftext` and `endoftext` tokens on each sample? That got it to finally return my prompt,
but no generated text.
  2) Did that fact that there were multiple newlines in a row in the reports (for positioning) cause it to 
over-learn newlines? Not really...now it just returned `'\n <my prompt>\n`
  3) Did Gpt2 not handle more than 2 spaces in a row well? This was the first one to return something longer than
the initial prompt, but it was still terribly wrong: 

  ```\n checkerboard spider.com/wp-content/uploads/2018/10/checkerboard- spider.com/wp-content/uploads/2018/10/checkerboard- spider.com/wp-content/uploads/2018/10/checkerboard- spider.com/wp-content/uploads/2018/10/checkerboard- spider.com/wp-content/uploads/2018/10/checkerboard- spider.com/wp-content/uploads/2018/10/checkerboard- spider.com/wp-content/uploads/2018/10/checkerboard- spider.com/wp-content/uploads/2018/10/checkerboard- spider.com/wp-content/uploads/2018/10/checkerboard- spider.com/wp-content/uploads/2018/10/checkerboard- spider.com/wp-content/uploads/2018/10/checkerboard- spider.com/wp-content/uploads/2018/10/checkerboard- spider.com/wp-content/uploads/2018/10/checkerboard- spider.com/wp-content/uploads/2018/10/checkerboard- spider.com/wp-content/uploads/2018/10/checkerboard- spider.com/wp-content/uploads/2018/10/checkerboard- spider.com/wp-content/uploads/2018/10/checkerboard- spider.com/wp-content/uploads/2018/10/checkerboard- spider.com/wp-content/uploads/2018/10/checkerboard- spider.com/```

At this point, I was convinced there was a problem with the input to the system, so I re-ran the Prefect parsing job
to do the following: 
  1) remove all the code snippets like in the LSTM earlier
  2) replace all multi-character whitespaces (newlines, tabs, spaces, etc) with a single space
  3) replace everything that looked like a URL with a single space

On the plus side, after doing that the model trained *MUCH* faster than previously, which made me think I was making
progress (spoiler: this whole line of troubleshooting is a red herring). 

On the minus side, the result of doing all that work was this:

```'\n checkerboard spider.com/blog/2018/05/checkerboard-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-sophisticated-soph'```

Which is hilarious, but wrong. 

If I chopped out *all* of the things that even look vaguely like a wordpress URL, the result becomes:

```'\n checkerboard spider. This is the first time we have seen this type of malware being used in the wild, and it’s the first time we have seen this type of malware being used in the wild, and it’s the first time we have seen this type of malware being used in the wild, and it’s the first time we have seen this type of malware being used in the wild, and it’s the first time we have seen this type of malware being used in the wild, and it’s the first time we have seen this type of malware being used in the wild, and it’s the first time we have seen this type of malware being used in the wild, and it’s the first time we have seen this type of malware being used in the wild, and it’s the first time we have seen this type of malware being used in the wild, and it’s the first time we have seen this type of malware being used in the wild, and it’s the first time we have seen this type of malware being used in the wild, and it’s the first time we have seen this type of malware being used in the wild, and it’s the first time we have seen this type of malware being used in the wild, and it’s the first time we have seen this type of malware being used in the wild, and it’s the first time we have seen this type of malware being used in the wild, and it’s the first time we have seen this type of malware being used in the wild, and it’s the first time we have seen this type of malware being used in the wild, and it’s the first time we have seen this type of malware being used in the wild,```

Which is doing the same thing, but not quite as funny as the one earlier.

At this point, I started looking more closely at the warnings that been printed on some previous steps, and 
I  noticed that in the Tokenization step, it was printing an error that said:

```Token indices sequence length is longer than the specified maximum sequence length for this model (26837 > 1024). Running this sequence through the model will result in indexing errors```

I wondered if the Tokenization step was assuming that no document would have more than 1024 words/tokens in them, and 
was having problems when my documents were routinely longer than that. So, I re-did the input reading step, to cut each 
document into at most 1024 words (including the previous space, url, etc removals). That led to:

```\n checkerboard spider.exe, which is used to download and execute a payload from the C&C server. The payload is then sent to the C&C server using the HTTP GET request. The payload is then sent to the C&C server using the HTTP GET request. The payload is then sent to the C&C server using the HTTP GET request. The payload is then sent to the C&C server using the HTTP GET request. The payload is then sent to the C&C server using the HTTP GET request. The payload is then sent to the C&C server using the HTTP GET request. The payload is then sent to the C&C server using the HTTP GET request. The payload is then sent to the C&C server using the HTTP GET request. The payload is then sent to the C&C server using the HTTP GET request. The payload is then sent to the C&C server using the HTTP GET request. The payload is then sent to the C&C server using the HTTP GET request. The payload is then sent to the C&C server using the HTTP GET request. The payload is then sent to the C&C server using the HTTP GET request. The payload is then sent to the C&C server using the HTTP GET request. The payload is then sent to the C&C server using the HTTP GET request. The payload is then sent to the C&C server using the HTTP GET request. The payload is then sent to the C&C server using the HTTP GET request. The payload is then sent to the C&C server using the HTTP GET request. The payload is then sent to the C&C server using the HTTP GET request. The payload is then sent to the C&C server using the HTTP GET request. The payload is then sent to the C&C server using the HTTP GET request. ```

This is...better. There's now two sentences, but it still locks onto a single phrase and repeats that endlessly. 

After digging a bit, I found out that there are a bunch of settings you can make on the *output* side of things. The
first one that I discovered was `no_repeat_ngram_size`. That settings says that the generation algorithm will track
n-grams of the size  you specify, and never repeat any given n-gram. For example, if you put a `no_repeat_ngram_size` of
2, then the generation might say "New York" *once*, but never again for that call to the text generation method. That
led to: 

```\n checkerboard spider.exe, which is used to download and execute a payload from the C&C server. The payload is then sent to a remote server, where it is decrypted and sent back to the attacker. Once the payload has been downloaded and executed, it will be executed in a new thread. This is the first time we have seen this type of malware being used in the wild. It is interesting to note that in this case, the malware does not contain any malicious code at all. Instead, we see that the attackers are using this malware as a lure to lure victims into clicking on the malicious link. Figure 1: Screenshot of a malicious downloader Figure 2: A screenshot of an infected web page Figure 3: An infected webpage Figure 4: Infected web pages Figure 5: The infected page is shown in Figure 6 Figure 7: Figure 8: Malicious PDF document Figure 9: PDF file containing malicious content Figure 10: Attacker’s web browser Figure 11: Spearphishing email with malicious attachment Figure 12: Attachment containing malware Figure 13: HTML document containing a link to an embedded malicious file Figure 14: Embedded malicious document in PDF format Figure 15: Link to embedded executable Figure 16: Exploitation method used by attackers to infect victims Figure 17: Targeted spearphish email sent from a compromised web server Figure 18: Credential harvesting technique used for phishing attacks Figure 19: Command and control (C2) communication between attackers and C2 servers Figure 20: HTTP POST request for a command to be sent Figure 21: User-Agent (HTTP/1.1) response Figure 22: Content-Length of the HTTP request Figure 23: Data sent by the command Figure 24: Encrypted data received Figure 25: MD5 hash of encoded data Figure 26: SHA-1 of decoded data (SHA-256) Figure 27: Hash of encrypted data in encrypted form Figure 28: XOR-encrypted data with a key of 0x5A```

This is a *LOT* better than before. It's still not great, and it's getting stuck in Figures.

So, that led to an obvious question: were all my attempts at removing spaces, urls, etc, unnecessary, and I just needed
to use `no_repeat_ngram_size` (and possibly keep the document size thing, since it removed an error). Trying that on
the initial dataset, I got:

```\n checkerboard spider.com/wp-includes/pomo/checker.php https://www.trendmicro. com/en_us/research/20/l/sidewinder-apt-group-continues-to-target-government-and-military-institutions-in-the-middle-east/ 1/12 2/4/2020 Sidewinders: APT group continues to target government and military institutions in the Middle East | TrendLabs Security Intelligence Blog Figure 1. Screenshot of the malicious document. Figure 2. A screenshot of a decoy document with a link to a malicious PDF file. The document has the following structure: Figure 3. An example of an email sent to an individual with the subject “Your email address will not be published.” Figure 4. Email sent by the individual to the email. This email was sent from the same address as the one used to send the spear-phishing email (Figure 5). Figure 5. Spearphished email from a victim with an attached PDF attachment. In this case, the attachment was a Word document, which contained an attachment that contained malicious code that would download and execute an executable file from hxxp://192.168.0.1:8080/download.exe Figure 6. Code that downloads and executes the downloaded executable. It is worth noting that this executable is not a legitimate executable, but rather a downloader that is downloaded and executed by a remote attacker. We believe that the attackers behind this campaign are using a custom toolkit that allows them to execute arbitrary code on the victim’s machine, as seen in Figure 7 and Figure 8. Conclusion The threat actor behind the campaign has been active for at least two years, and it is likely that they continue to be active as long as they are able to maintain access to victim networks. However, we do not have enough evidence to conclusively attribute this activity to any specific actor or group of threat actors. Based on our telemetry, it seems unlikely that any particular actor is responsible for this attack, or that there is any direct connection between this actor and the group behind it. Further research is needed to determine who is behind these attacks and how they operate. Indicators of Compromise (IoCs) The following IoCs can be used as indicators of compromise in order to identify potential victims of interest: • C&C server IP address • IP addresses used for command and control (C2) • Domain names used in spear phishing emails • Malicious domains used by attackers to deliver malicious payloads • Email addresses and URLs that redirect victims to malicious domains • URL short URL paths that lead to legitimate websites (e.g., www.google[.]com, google.co.uk, etc.) • MD5 hashes of malicious files • SHA-256 hash of files that have been uploaded to VirusTotal (SHA-1 hashes) We have also identified a number of other domains that are associated with this group. These domains are linked to other groups that we believe are related to this threat group, such as DarkHotel and DarkComet. While we have not found any evidence of direct links between these groups,```

Which implies that I really didn't need to do all that filtering, that in fact the *generation* side of GPT2 was where
all the problems had been. So, I started to investigate other options on the text generation side of things. I found 
a [blog post from HuggingFace](https://huggingface.co/blog/how-to-generate) that went into great detail about this,
and in particular looked at 4 generation options:

  * `num_beams`: a "beam" is a set of words in a tree of possible words following a given word. Using beams, the algorithm picks the *beam* with the highest probability, rather than the individual next word. That makes some sense, since it's more likely to make coherent sentences. It does, however, have the repeating problem I've seen here. 
  * `Temperature` is a measure of randomness that gets added to the sampling, but apparently doesn't help much in beaming, since it'll just keep picking the same high-probability beams. 
  * `early_stopping` allows the algorithm to stop before the max or min length if it's used up all its possible outputs, which helps avoid repetition.
  * `no_repeat_ngram_size` is a way to stop it from repeating words. It means that the algorithm keeps ngrams that have already been used in the text generation, and ensures they're never used again in that generation pass. That's great for breaking out of repetition, but it means that names, for example, will never be used again, which is a problem. (In the blog example, `New York` would trigger a 2-gram, and so would only ever be used once in a text generation, which may not be what you want if you want something about New York.

I'd already tried the ngram setting, but the blog post also mentioned a method that seemed to work better than 
Beaming (the `num_beams` parameter above): `top-k` and `top-p` sampling. 

`Top-k` Sampling is a method for deciding what the next word will be when generating text. For Top-k, you limit the
possible set of choices to the `k` most popular words. The advantage of doing it this way is that you avoid the generation
algorithm launching into weird territory from very general words. The disadvantage of this method is that some words
very naturally have lots of words that could follow them (like "the") and Top-k hard-cuts those off.

`Top-p` sampling instead limits the words that you can pick to the ones that fall in the top percentage of likely
next words, the percentage cutoff defined by `p`. So, if a word has many possible choices, and you set `p` to 90%, you
will only cut off the 10% *least* likely words. 

So, starting from the full, unfiltered input set, I decided to try the following settings:
 * temperature = 0.7
 * top_p sampling of 92%
 * top_k sampling disabled

The result for that attempt was:
```'\n checkerboard spider - a new open-source toolset with a command-line interface that can be found in the open-source Firefox spyware webshells, see the list of “Spyware” here. Most of the tools are not available on the open-source web server. The webshells are built on top of the popular webshells, such as the “Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail.Mail```

which gets off to a great start, but lands somewhere super-weird. There is one last setting the blog post mentioned,
which is `num_return_sequences`. The idea here is that num_return_sequences will allow the generation algorithm to 
track multiple threads of generation at once, to decide which *series* of words comes to the best generation likelyhood,
rather than treating them as words one-by-one.

[That worked *quite* well](https://gist.github.com/g-clef/8ea6b388a931f570615fd55b3fbbefe3). Well enough that I called 
it a success and stopped. 

#### Special stuff for GPT2

In short, here's what I had to do outside of the FastAI tutorial to get GPT2 to work with my setup:
 * Pre-process the PDFs to turn them into text files. 
 * Read the text files into lines of a Pandas dataframe, splitting them into < 1024-word documents
 * Pre-tokenize all the documents so they'd only have to be parsed by the tokenizer once
 * When generating text, use `temperature`, multiple return sequences, and `top_p` sampling to avoid the generation 
falling into a repeated pattern.

