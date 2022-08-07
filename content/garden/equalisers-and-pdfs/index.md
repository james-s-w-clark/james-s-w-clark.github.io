---
title: "How PDFs make your headphones sound better 🎧"
date: 2022-07-20
lastmod: 2022-08-07
draft: false
garden_tags: ["audio, random"]
summary: "An intro to equalisers, and two ways EQ configuration can be extracted from PDFs"
status: "growing"
---

This blog post will be a mishmash of audio equalisers and PDF text extraction.

Equalisers (EQ) let you adjust your audio to your liking. They can help all headphones try to achieve a common sound, such as the Harman Curve. There's loads you can read around the topic, but to keep things practical you could just give it a try and see if things "sound better".[^1] 

[^1]: The usage/installation guide at [AutoEq](https://github.com/jaakkopasanen/AutoEq#usage) is pretty good. The Peace GUI makes it super easy to download and use the AutoEq settings.

The user "Oratory1990" has [measured a huge number of headphones](https://www.reddit.com/r/oratory1990/wiki/index/list_of_presets/) and has given equaliser settings you can try out. In the case of used Beyerdynamic DT 770 Pros, EQ will take their "preference rating" from 79% to 100% - basically, statistically speaking, people will like them more with EQ.

Recently the measurements and fine-tunings were adjusted for my headphones, so off I went to figure out how to use the new configuration...

# Creating a parametric EQ for Peace / Equaliser APO

In my case, I want to use the new config for Sennheiser HD 660s - see [this dropbox pdf](https://www.dropbox.com/s/s4isno09gt20ozf/Sennheiser%20HD660S.pdf?dl=0):

{{< figure src="filter_settings.png" width="100%" >}}

To make this config simply and correctly, we can use an exisiting parametric EQ text file from [AutoEq](https://github.com/jaakkopasanen/AutoEq/blob/master/results/oratory1990/harman_over-ear_2018/Sennheiser%20HD%20660%20S/Sennheiser%20HD%20660%20S%20ParametricEQ.txt). You can see the number of filters, where the peak of each filter is centred (30Hz), how loudness changes around that frequency (6.5 dB) and the strength of the curve (Q 0.37) - as well as the shape of the curve (PK):

```bash
Preamp: -7.0 dB
Filter 1: ON PK Fc 30 Hz Gain 6.5 dB Q 0.37
Filter 2: ON PK Fc 181 Hz Gain -3.8 dB Q 0.54
Filter 3: ON PK Fc 1351 Hz Gain -3.3 dB Q 1.33
...
```

We can see that "PEAK" is abbreviated to "PK" - so we can guess that the low & high shelves from the diagram will be LS & HS respectively. Copying the preamp gain over and adjusting the numbers gets us:

```bash
Preamp: -8.4 dB
Filter 1: ON PK Fc 34 Hz Gain 3.0 dB Q 1.5
Filter 2: ON LS Fc 100 Hz Gain 5.5 dB Q 0.71
Filter 3: ON PK Fc 200 Hz Gain -2.5 dB Q 0.8
Filter 4: ON PK Fc 1330 Hz Gain -2.5 dB Q 1.8
Filter 5: ON HS Fc 1700 Hz Gain 4.0 dB Q 0.71
Filter 6: ON PK Fc 3300 Hz Gain -2.1 dB Q 1.7
Filter 7: ON PK Fc 5520 Hz Gain -6.4 dB Q 3.8
Filter 8: ON PK Fc 8000 Hz Gain  3.0 dB Q 1.4
Filter 9: ON HS Fc 10000 Hz Gain -2.0 dB Q 0.71
Filter 10: OFF
```

I use Peace for Windows, so I need to import the config this way:

{{< figure src="peace_import.png" width="100%" >}}

... and don't forget to save the config (feel free to choose your favourite colour icon!):

{{< figure src="peace_save.png" width="80%" >}}

we should see that the profile is active in Peace, and in the taskbar:

{{< figure src="peace_taskbar.png" width="30%" >}}


# Are there lazier ways to get good EQ from these numbers?

Yep, like mentioned above there’s [AutoEQ](https://github.com/jaakkopasanen/AutoEq). The readme is good, so you can just follow that if you want more info on how to use it.

How does AutoEQ generate the EQ files?

There is a crawler python script, which:

- [scans a reddit wiki](https://github.com/jaakkopasanen/AutoEq/blob/master/measurements/oratory1990/oratory1990_crawler.py#L47)
- [renders PDFs into images](https://github.com/jaakkopasanen/AutoEq/blob/master/measurements/oratory1990/oratory1990_crawler.py#L156)
- [parses the frequency graph into response curve data](https://github.com/jaakkopasanen/AutoEq/blob/master/measurements/oratory1990/oratory1990_crawler.py#L81) by [reading the coloured data points](https://github.com/jaakkopasanen/AutoEq/blob/master/measurements/oratory1990/oratory1990_crawler.py#L128)

As you might guess, it uses some PDF libraries, and even checks the text in the image. Instead of getting the table of data, it uses the graph to generate data. The nice thing about this is that you can independently work out graphical/parametric EQs to reach different target audio curves.

# How about automatically reading the text from the tables?

I’m going to tell you a bit about how data is expressed in PDFs, and how we can explore PDFs to find out how we can extract useful data. You can follow along on your own PC and see what's going on with PDFs if you like.

Let’s see how information is expressed in PDFs:

- [Get a PDF to examine](https://www.reddit.com/r/oratory1990/wiki/index/list_of_presets/)
    - e.g. [1More earphones](https://www.dropbox.com/s/90dggt4jel4jjgc/1More%201M301%20Single%20Driver.pdf?dl=0)
- [get the latest PDFBox debugger](https://pdfbox.apache.org/download.html) 
    - install Java if you don't have it
- run `java -jar debugger.jar`
- Open the PDF file in the debugger that pops up. It will look like:

{{< figure src="pdf_debugger.png" width="100%" >}}

If we view the contents of the file, we can view either the plaintext or encoded data stream. There’s a bunch of **Stuff**, but let’s look for BT (Begin Text) / ET (End Text) and see what we can read.

{{< figure src="pdf_decoded_text.png" width="100%" >}}

`(m) 33 (e) -44 (a) -44 (su) -44 (r) 33 (e) -44 (d) -44 ( ) -22 (o) -44 (n)` -  "measured on" - that text does indeed appear in the document. What's with all the numbers, and having the letters in brackets? The numbers are the codepoints of glyphs+unicodes in fonts embedded in the PDF - you can see stuff like this in the font resources:

{{< figure src="pdf_code_glyph_unicode.png" width="100%" >}}

Note that whilst the letters for "measured on" were at the top of the decoded stream, that doesn't imply where the text will be on the document. The location is defined by coordinates, like 33.06, 44.38977.

Given that I can search the PDF in a PDF viewer for “measured on” and find it, and that I can also the  “Band 1” and find it - I’m pretty confident the text is expressed “plainly” in the plain text (as something like 2(b) 1(a)...). 

There’s 225 “BT”s in the document. I had a flick through all of them and didn’t see something that was obviously our data. BW is the most unique short phrase in the document - can we find that? Searching for (W) gives no results. How can searches with a commercial PDF viewer find results for W? As I write this, I'm not so sure.

We can take a step back and try some higher-level tools to keep things simple. PDFbox comes with a CLI - we can try a few commands:

- `java -jar .\pdfbox-app-3.0.0-alpha3.jar` - shows us there's an "export:text" option
- `java -jar .\pdfbox-app-3.0.0-alpha3.jar export:text --input=pdf.pdf` - gives us some interesting plain-text output:

```bash
...
Filter Type Frequency Gain Q-Factor BW
Band 1 PEAK 80 Hz -9,6 dB 0,4 3,02 Adjust gain of band 2 to preference (bass)
Band 2 LOW_SHELF 110 Hz 10,0 dB 0,71 1,89 Deviation from Target Adjust gain of band 7 to preference (airiness)
Band 3 PEAK 300 Hz -2,5 dB 1,0 1,39 Before EQ After EQ
...
Band 10
...
Preamp gain:
-9,0 dB
```

{{< figure src="1more_table.png" width="100%" >}}

It looks like we’ve got everthing we’d want! And look at that preference rating jump: 58 to 95. Those earphones will sound twice as good with this EQ 😉

If you’re interested in extracting this information and playing with how PDFs work, I recommend you to have a deeper look with some of these tools. It looks like you could get something working quickly with PyPDF2, to extract this text and transform it into an EQ that Peace could use. No measuring of graphs and generating of configuration needed - you can just extract the provided configuration.

# Personalising EQ

We mentioned Harman curves, preference ratings, EQs and stuff - but every human is different, right? Generally, the Harman target will sound good to lots of people - but some people love bass, and some people hate sibilance. A good EQ can be tweaked to personal preference.

Here's some notes from Oratory, then we can do a little experiment:

{{< figure src="oratory_band_preference.png" width="100%" >}}

---

{{< figure src="oratory_db_tweaks.png" width="100%" >}}

## Sibilance - electric drum machine

I listened to "Paper Love" by Allie X, and the hi-hat(?) on the electric drum machine was a little too loud for me - we can make it less of a focus if we want to.

To figure out where the sounds we want to change mostly lie, we can try making everything else quiet:

{{< figure src="eq_isolate.png" width="50%" >}}

Tweaking the dials further, I figured out that -3dB at 8k and 10k was good for me. Vocals might be a *tiny* bit squashed, but the song was much easier to listen to. Optimising for listening time can be a fair goal.

I saved this config as "660s_oratory_2022-07-15_relaxed_his", and with a different colour icon.

If you can't hear any change from moving the dials/pre-amp, you might need to restart your PC or check your Equaliser APO installation.

## More bass/sub-bass

This time we can try a different song. Choose whatever you like, or "Regulars" by Allie X (I do listen to other stuff, but was going through all the albums that day!). Try setting everything down and raise the sub-bass frequency:

{{< figure src="eq_only_subbass.png" width="50%" >}}

With too much of this, there's mostly a kick drum(?) and muffled voices. Turn everything back else to the EQ you want and then turn the lowest frequency down - it feels a bit like you've lost the "body"/substance of the song.

I'm sticking with the default provided by the EQ.

## Bass

I tried similar for 100 Hz, so probably "bass" frequencies. If I took it too high, other sounds seemed to get really quiet. I'm using a decent DAC (Schiit Modi) and amp (DIY O2), so I'm not sure what's going on. But I'm OK with Oratory's default, it's got enough bass for me.

Again, I'm sticking with the default provided by the EQ.

# Conclusion

We're all different. Audio is down to preference. You might like EQ, you might not. Why not give it a try? It's free and could add even more enjoyment to your music listening.

# Extra notes

Here's an interesting aside: sometimes PDFs are very weird about how they show text. I’ve seen what looks like totally normal documents, but on further inspection they have no actual letters in. They just have a bunch of lines and curves that looked like letters. Documents like this have to be rendered to an image and OCR’d to extract the text!
The heuristic for figuring out whether to do that is fairly simple (but expensive, so don't do it by default): look for a ridiculous number of lines/curves, or like 99% of the PDF operations being line/curve draws.

Here’s an example of line draws and curves in one of the docs we looked at today:

{{< figure src="pdf_lines_curves.png" width="100%" >}}

If you want to read more about PDFs, [this 2008 PDF .7 spec by Adobe](https://opensource.adobe.com/dc-acrobat-sdk-docs/standards/pdfstandards/pdf/PDF32000_2008.pdf) was the top Google result. It's more than 750 pages long. I spent a lot of time reading that thing in my first Software job 😬