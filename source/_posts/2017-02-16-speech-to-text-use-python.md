---
title: 使用python来语音转文字

date: 2017-02-16 05:15:31
tags:
  - speech_recognition
---

今天在 `youtube` 上看到 [[Open Jarvis] 如何讓Python 自動將語音轉譯成文字?](https://www.youtube.com/watch?v=31DZfkYRvI4), 就想到前段时间想
把没有字幕的视频里面的语音转成文字, 这样好理解一些(英语渣的苦,谁渣谁知道).
找了半天都是要申请 `api key` 还有各种限制, 比如收费(哈哈).

## 安装
### ffmpeg
官网: [http://ffmpeg.org/](http://ffmpeg.org/)    
这里用它来提取视频里面的音频.

    brew install ffmpeg

### SpeechRecognition
[SpeechRecognition](https://pypi.python.org/pypi/SpeechRecognition/)    
它是把在线的,离线的,各种aip接口集合在一起,然后提供统一的接口.
因为我不喜欢要申请key的,所以只能用离线的 `CMU Sphinx`.

先去 [CMUSphinx](http://cmusphinx.sourceforge.net/wiki/download)下载安装 `sphinxbase` `pocketsphinx`, 可以看 [cmusphinx/pocketsphinx](https://github.com/cmusphinx/pocketsphinx).
在安装它们之前还要安装 `swig`,不然会报错.

    brew install swig

都安装好后在

    pip install pocketsphinx


## 提取音频
比如我要提取 `[das-0091-introduction-to-computation-4k.mp4](https://www.destroyallsoftware.com/screencasts)` 的音频为 `test.wav`    

    ffmpeg -i das-0091-introduction-to-computation-4k.mp4 -vn test.wav

因为 `AudioFile` 接口只支持 `WAV/AIFF/FLAC`, 所以用的是 `wav`.


## 提取文字
可以看官方的例子 [audio_transcribe.py](https://github.com/Uberi/speech_recognition/blob/master/examples/audio_transcribe.py) 很简单.

    import os
    import speech_recognition as sr

    r = sr.Recognizer()
    with sr.AudioFile('test.wav') as source:
    audio = r.record(source)

    r.recognize_sphinx(audio)

输出如下:    


    this series is going to cover competition and i think we should begin by just laying out the topics are going to mention along the way so we have a sense of where we're going the first of those topics is the radical simplicity of computation it turns out that all the complexity of our computers and programming languages and operating systems is comp complexity that we have added it is not fundamental to computation and to see that we're going to look both at the turn machine and abby lana calculus which are the two most well known models of computation were these the two most well known abstract models both of these are normally talk with the very mathematical kind of terminology indication lot of greek letters and so on but we're just going to use python code because python is aline was any programmer can understand pretty easily and that wall hours to get at least a high level understanding of what's going on inside of the systems the next topic that we're going to talk about is the limits of computation specifically be holding problem which is an example of an undesirable problem at the computer science be holding problems as they did really quickly is the problem of writing a function let's call a halt it takes another function of scott after the decides whether half will terminate we're not itself will terminate eventually then hold your return troops itself wolford sample would forever than a halt to return faults and it's easy to state that problem as i just did it but you cannot write this launch and no programming language can express this function no computational system can express his function at the highly non obvious result but there are rigorous mathematical proof so this going back to the nineteen thirty's and they have held up for a year's both in theory and practice so will see why that's true are these the high level sketch of why that's true and some of the implications of it for the rest of computation we're also going to see the structure of computation specifically the idea of trying equivalents which tells us that the turing machine and amanda calculus are both capable of answering the same questions any question that one of them can answer the other cancer and it turns out to this is true of our real world computers as well including the laptop and i'm recording this on if my laptop unanswered question and so could turn machine and this is extremely surprising given how simple turn machines are exactly first look at them you're not going to believe that their actual general purpose computer system but it turns out that in fact they are because of turner problems which once again as a rigorous mathematical proof is going back to the nineteen eighties excuse me hit eighty years ago the nineteen thirty's a related idea that will talk about is the chon ski hierarchy of computational systems and it turns out that this turn of quo blood type of system is only the most powerful type of computation there are four levels and as hierarchy and they began with the weakest which is a bullet to what we called regular depressions the next level is what you would need if you wanted to recognize python code mi you want to decide whether as spring is valid python or is not about python the next levels which would need if you wanted to recognize as c. plus plus code and this distinction is very important in fact python was intentionally designed to require less complexity in the top additional system that recognizes that most programming languages have this level of complex including for example a job as script and one of the reasons the python is so we see the lord and revisit potentially was designed to require less complexity finally the last level in this hierarchy is the turner problem level which contains trainers shane solana calculus my laptop and so on and this hierarchy is first of all amazing just as relates these things that seem unrelated if you haven't learned as yet but that's not even the most amazing thing about it the most amazing thing is it known tom speed created this hierarchy when he was studying linguistics he was studying natural languages like english and he establishes different levels of linguistic complexity which computer scientists then took the news for all kinds of things including programming languages but also categorizing finite state machines which fall into these different categories in different ways and will see all of these things are more detail as we go that's all i want to say about this introduction next time we're going to pick up trade machines were gonna ride that simulator that could be wrapped hemlines a code and take about ten minutes of soul be quite easy to write so i'll see you next time for training sheets


## 总结
例子很简单,主要时间花在安装包上面,提取出来的文字虽然还有错误, 但大体意思还是不难理解的.不知道其他在线的接口的效果怎么样,我个人觉得正确率应该会高些,有兴趣的可以自己试下.
