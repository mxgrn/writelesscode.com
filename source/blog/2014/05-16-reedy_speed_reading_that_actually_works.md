---
title: "Reedy: speed reading that actually works "
created_at: 2014-05-16
kind: article
tags: [productivity]
excerpt: Reedy is a tool that allows you to increase the speed of reading by 3-5 times. And it's a very solid one.
---

*This is a translation of an article in Russian published on [Habrahabr](http://habrahabr.ru/post/220775/).*

![Reedy, advanced speed reading tool](http://writelesscode.com/images/2014-05-16-1.png)

Recently, there has been an increasing interest in [RSVP](http://en.wikipedia.org/wiki/Rapid_Serial_Visual_Presentation)-based speed reading techniques. However, most applications and tools on this topic, on a closer acquaintance, are not very comfortable for daily use. Even though, in potential, they can help significantly save time and consume information more rapidly. My friend OlegCherr decided to modify this technology, creating an implementation suitable for prolonged use, and I am helping him in this. After two months of constant reading, testing, researching and making improvements, I decided to write a detailed article about the result, as I dare to believe it shall be interesting to others. Currently, it is implemented in the form of an extension to Chrome, and is called Reedy. The Android application is on the way.

## Ability to view the full text

One of the main problems when reading via speed reading techniques is losing context. It is difficult to understand, in which part of the text you are at the moment, and it is difficult to re-read the last words or sentences.

Therefore, one of the main differences of Reedy from other implementations is the ability to view the text context during the pause. This function is already found in some Android-apps, however it has been best implemented in fastreader (from citriccode), where it was borrowed from, and then it underwent a series of improvements, the main of which are:

* It is possible to toggle between sentences by scrolling the mouse wheel. Periodically, however, you forget that you are in an application for fast reading and you start to read the text as usual, just as if it was a page of text, which you keep scrolling down. Don't get carried away :)
* A comfortable scrollbar, giving an idea of ​​the current location in the text

![Reedy, showing text context](http://writelesscode.com/images/2014-05-16-2.png)

At any time, with one click or by pressing a hotkey you can pause reading, see the text before and after the current word, move back to a word or to a sentence, etc. The progress bar and the remaining time indicator (based on the the current rate) also help getting lost in the text. In addition to this, during the pause, many other functions become available: changing the subject, background, text position on the page, and some others.

## Text processing

Another Reedy's feature is the text analyzer, allowing to display common speech patterns together (thus not tearing them into separate meaningless units), as well as carrying out “smart” deceleration – on difficult words and punctuations the rate decreases and at the end of the sentences a longer pause is added. Words which are too long, on the contrary, are divided into several parts.

The analyzer consists of four levels, each of which performs a specific task:

1. The first level performs basic character recognition in the text, identifies groups of similar characters.
2. The second level defines minimal indivisible tokens. This is when the identification of common structures is produced, such as:

    * Words with a hyphen and other inclusions: dark-grey, d'Artagnan
    * Links: www.example.com/path
    * Phone numbers: +7 960 123-45-67, (815 2) 123456
    * Others monolithic structures

3. The third level recognizes compound structures, such as a surname with initials.
4. In the last phase the analysis of the punctuation is carried out. The desired characters “cling” to the desired word (for example, a dash will be attached to the next word, if a direct speech has started, otherwise it will be attached to the current word). The beginning and the end of each sentence are defined.

![Reedy, text processor](http://writelesscode.com/images/2014-05-16-3.png)

It should also be noted, that before getting into the parser, the text is pre-processed, where obvious syntax errors are fixed (such as extra spaces, dangling apostrophes or brackets, multiple exclamation and question marks and other debris, which prevent comfortable speed reading). If you want to read a material in which such structures are an important component, you can always deactivate the purification and analysis functions (Settings ›Text ›Entities analyzer). The expansion options are very flexible to configure.

## Focus mode
Of course, in Reedy the focus mode is present, too, in which your eyes will be fixed on a single letter, color-highlighted. This letter is the most comfortable place to focus your eyes, in order to quickly and accurately recognize the current word.

Another implementation feature is that the highlighted character can be shifted in case it falls not on a letter or digit, but, say, on a hyphen inside a word. The algorithm will try to shift the focus closer to the beginning of the word if there is an alphabetical character present, or otherwise the focus will be shifted closer to the end of the word.

## Smooth acceleration
The smooth acceleration function makes it much easier to adapt to high and to ultra-high speed reading. The initial speed is set lower than the targeted one, and then gradually increases. The function has some features, which we chose to implement after using the tool for some time:

* At the first launch, the reading speed is set to half of the targeted one. However if you click on the pause to read the previous segment and then start reading again, in this case it will be more comfortable not to start at a half speed, but with a speed a bit faster. In Reedy this figure is set to 60% of the targeted value.
* If you are reading at a speed of 700 wpm (words per minute), then you will not be comfortable for long, to observe a smooth acceleration from 350 to 700. This is why Reedy increases the speed not in a linear way, but using a sine function: first it accelerates faster and then slower and smoother. After long experimentation, this method of acceleration proved to be the most comfortable.

## Showing extension of the text
There's also one experimental feature In Reedy.  In the settings it is called an extension of the text. It includes showing a few words that follow the current visible. The idea was developed in order to improve the readability of complex texts.

In the initial version not only the following words were shown, but the previous ones as well, in exactly the same color as the current word. The result was a “running line effect”. However, with a few experiments, we found out that for the best text digestion, it was necessary to show only the subsequent words. Additionally, they should have much less contrast, so as not to interfere with the recognition of the current word.

![Reedy, extension of phrase](http://writelesscode.com/images/2014-05-16-4.gif)

Also, an interesting point is that the subsequent words are limited to those of the current sentence. This means that, if the word is the last one in a sentence, then no extension of the text is shown. This helps to improve the understanding of the reading snippets. Most important is to keep the attention on the first word, and not to let the focus “slip forward”. If that happens then, perhaps, it makes sense to increase the reading rate.

As the result, we achieved a better consumption of the text by more efficient usage of the peripheral vision. This feature can be especially useful to people that read a few words at a time during their normal reading, i.e. with already well developed skills of the conventional speed reading. Nevertheless, this may not be for everyone. So by default, this option is disabled. You can enable it in the settings: Display -> Continued text.

## Frequently asked questions
Does such a breakneck speed affect the digestion of information?
It depends on the skills. For a novice, maybe only a 50% speed increase is possible without affecting the quality of understanding. But if you use Reedy regularly and get used to it, the speed of recognition and understanding increases — the information gets absorbed more quickly. Overclock your brain!


### Why would we need to read so fast?
In first place, to save time. To consume information quickly. And in the situations, when you can sacrifice a bit of a quality of understanding, you can manage to get an even bigger gain in speed. For example, to read the news, you can specially put a speed at which information is acquired with some loss — the general idea will stay clear.

### What to do if I face problems with the understanding of the text?
Maybe you press the pause button too often in order to re-read the previous passage, or your imagination does not keep pace.

* Make sure that you do not articulate, make involuntary movements of the lips and tongue while reading a text. Try to perceive the text with your eyes; do not pronounce it to yourself. By the way, speed reading teaches you that. Because the higher the speed, the more difficult it becomes to keep up pronouncing the words. So eventually the pronunciation will be gone, even if no effort was specifically made to achieve this.
* Start reading at a speed 300 words per minute max. Read in a quiet environment. Read simple texts — articles on general subjects, fiction or even texts that you have read before. After some time (maybe in five minutes, maybe in a day) you will notice, that the text at the current rate is perceived quite easily, and you rarely stop reading to re-read the previous passage. Now you may increase the speed by 50 words per minute.
* Pick the right speed. You should neither increase nor decrease the speed while reading. Determine “your” speed. Please note that the texts of different complexity may require different speed. The more complex the material, the more time you need for comprehension of the part that you have read, and the lower you should set the reading speed.
* On unknown words, you will most likely need to pause in order to understand their meaning. But this is true also for the ordinary reading.
* Adjust Reedy for maximum comfort. Pick a comfortable font size and text position. During daytime, under direct sunlight or on a glossy screen surface, it is better to use a light screen mode. In the evening it is recommended to turn on the dark screen mode, which puts less strain on the eyes.

### Is it possible to speed-read technical texts?
Yes, although it will only be possible to do so on portions of plain text, and probably, on lower speed, than usual. Naturally, formulas, codes, graphics and others will have to be watched as usual.

### Is it suitable for fiction?
Most definitely it is. The imagery is perceived more lively and colorful. You can say that the text turns into a movie. However, sometimes the opposite occurs when images are perceived worse and it becomes impossible to enjoy the material. But if you are lucky enough and posses fast creative thinking, then reading fiction will become much more exciting for you.

### How to solve the problem with blinking?
Do not forget to blink. This applies not only to reading. Prevent the drying of the surface of the eyes. Generally, blinking — it is a very quick process. Usually there is no complexity with blinking even while reading with a speed of 1000 wpm. But if there are difficulties, especially at high and ultra-high speed, it is recommended that somewhere once per minute to stop and squeeze your eyes shut for a few seconds. You can also try to have your eyes blinking in turns :) Well, maybe it is more advisable to simply lower the speed to a level at which there are no problems with blinking.

### How to read a PDF?
If the PDF file is not copy protected, you can open it in a Chrome browser, select the text that you want to read and then in the shortcut menu, select “Run Reedy”. This combination has a lot of bugs, though, so you need to select the text more accurately. It is desirable that the text is contained on one page.

### How to read anything?
Reedy has an offline mode — you can copy any text, then paste it in Reedy.

## Conclusion
With some practice, you can read at a speed of 1100 words per minute (this comes from personal experience). First of all, this applies to news or simple articles, in which case we talk about saving time by 3–5 times! More complex texts are quite possible to read at 500–700 wpm (at least twice the usual speed). You can train yourself to read very quickly even without making much effort, literally for a week or two, just gradually increase the number of words per minute.

To start reading you need to select a text or a part from a page (you can select a DOM-element). You can run it from the context menu, with the hotkey (by default Alt+S) or from the extension menu — use any method, which is convenient for you.

Reedy has a fully open source code and is licensed under the terms of GNU GPL v2.

You can install the extension from the [Chrome Web Store](https://chrome.google.com/webstore/detail/reedy/ihbdojmggkmjbhfflnchljfkgdhokffj).

**Happy reading!**
