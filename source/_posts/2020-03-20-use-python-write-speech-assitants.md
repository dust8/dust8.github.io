---
title: 使用 python 写一个语音助手
date: 2020-03-20 05:15:31
tags:
  - python
  - speech
  - assitants
  - 语音助手
---

前几天看到老外用 `speechrecognition` 写了一个语音助手, 感觉既简单又有趣,
而且我前几年用这个库做个语音转文字. 去年还买了本 `<<自然语言处理实战(聊天机器人技术原理与应用)>>` 的书,
还没翻过几页. 正好把这个冲动的结果给利用上.

源码: [xiaohui](https://github.com/dust8/xiaohui)

`xiaohui` 是个面向任务的对话系统.  
可以查时间,打开程序, 搜索网页等任务. 具体任务可以看 `xiaohui/data/nlu.md` 里面的内容.
还可以自己定制任务, 对应的也需要定制 `xiaohui/actins.py` 的执行动作.

## 主要模块

![主要模块](/blog/assert/2020-03-20.png)

`NLU` 模块我用的是 `fuzzywuzzy` 库来计算准确度的,简单的判断 2 个字符串的相似度. 没有用序列标注来训练和识别意图和槽值. 任务可以看 `xiaohui/data/nlu.md` 里面的内容.用它来和用户输入的语句做相似度对比,取最大的置信度. 这里的格式参考了 `Rasa`.

```markdown
## intent:greet

- 你好
- 您好
- 你叫什么名字

## intent:goodbye

- 再见
- 退出

## intent:get_time

- 现在几点
- 几点了

## intent:open_program

- 打开[记事本](program)
- 打开[google chrome](program)
- 打开[腾讯视频](program)
- 打开[百度网盘](program)

## intent:search

- 搜索[python](keyword)
- 搜索[新闻](keyword)
- 搜索[上海](keyword)
```

打开程序任务是查询系统的开始菜单里面的快捷方式, 要完整匹配才能打开.

```python
class ActionOpen_program(Action):
    def run(self):
        programe_name = self.slots[0]["value"]
        programe_exe = lnk.PROGRAMS.get(programe_name)
        if programe_exe:
            subprocess.call([programe_exe])
        else:
            utter_message(f"未找到程序{programe_name}")
```

搜索就是简单的打开浏览器搜索关键字

```python
class ActionSearch(Action):
    def run(self):
        search = self.slots[0]["value"]
        url = f"https://www.baidu.com/s?ie=UTF-8&wd={search}"
        webbrowser.get().open(url)
```

## 语音识别和合成

试用了 `CMU Sphinx` 的离线, 效果太感人, 就看下 `Google Speech Recognitione` 的接口, 才发现不需要 `申请api` , 注意不是 `Google Cloud Speech API`.
利用 `google` 的语音识别和语音合成, 适配的是国内域名 `cn` , 所以不用担心用不了. 这里是利用的 `hotfix` 把代码给替换了, 把 `com` 替换成了 `cn` .

```python
import speech_recognition as sr

from .utils import hotfix

sr.Recognizer.recognize_google = hotfix.recognize_google
r = sr.Recognizer()
```

```python
def recognize_google(self, audio_data, key=None, language="en-US", show_all=False):
    """
    Performs speech recognition on ``audio_data`` (an ``AudioData`` instance), using the Google Speech Recognition API.

    The Google Speech Recognition API key is specified by ``key``. If not specified, it uses a generic key that works out of the box. This should generally be used for personal or testing purposes only, as it **may be revoked by Google at any time**.

    To obtain your own API key, simply following the steps on the `API Keys <http://www.chromium.org/developers/how-tos/api-keys>`__ page at the Chromium Developers site. In the Google Developers Console, Google Speech Recognition is listed as "Speech API".

    The recognition language is determined by ``language``, an RFC5646 language tag like ``"en-US"`` (US English) or ``"fr-FR"`` (International French), defaulting to US English. A list of supported language tags can be found in this `StackOverflow answer <http://stackoverflow.com/a/14302134>`__.

    Returns the most likely transcription if ``show_all`` is false (the default). Otherwise, returns the raw API response as a JSON dictionary.

    Raises a ``speech_recognition.UnknownValueError`` exception if the speech is unintelligible. Raises a ``speech_recognition.RequestError`` exception if the speech recognition operation failed, if the key isn't valid, or if there is no internet connection.
    """
    assert isinstance(audio_data, AudioData), "``audio_data`` must be audio data"
    assert key is None or isinstance(key, str), "``key`` must be ``None`` or a string"
    assert isinstance(language, str), "``language`` must be a string"

    flac_data = audio_data.get_flac_data(
        convert_rate=None
        if audio_data.sample_rate >= 8000
        else 8000,  # audio samples must be at least 8 kHz
        convert_width=2,  # audio samples must be 16-bit
    )
    if key is None:
        key = "AIzaSyBOti4mM-6x9WDnZIjIeyEU21OpBXqWBgw"
    url = "http://www.google.cn/speech-api/v2/recognize?{}".format(
        urlencode({"client": "chromium", "lang": language, "key": key,})
    )
    request = Request(
        url,
        data=flac_data,
        headers={
            "Content-Type": "audio/x-flac; rate={}".format(audio_data.sample_rate)
        },
    )

    # obtain audio transcription results
    try:
        response = urlopen(request, timeout=self.operation_timeout)
    except HTTPError as e:
        raise RequestError("recognition request failed: {}".format(e.reason))
    except URLError as e:
        raise RequestError("recognition connection failed: {}".format(e.reason))
    response_text = response.read().decode("utf-8")

    # ignore any blank blocks
    actual_result = []
    for line in response_text.split("\n"):
        if not line:
            continue
        result = json.loads(line)["result"]
        if len(result) != 0:
            actual_result = result[0]
            break

    # return results
    if show_all:
        return actual_result
    if (
        not isinstance(actual_result, dict)
        or len(actual_result.get("alternative", [])) == 0
    ):
        raise UnknownValueError()

    if "confidence" in actual_result["alternative"]:
        # return alternative with highest confidence score
        best_hypothesis = max(
            actual_result["alternative"],
            key=lambda alternative: alternative["confidence"],
        )
    else:
        # when there is no confidence available, we arbitrarily choose the first hypothesis.
        best_hypothesis = actual_result["alternative"][0]
    if "transcript" not in best_hypothesis:
        raise UnknownValueError()
    return best_hypothesis["transcript"]
```

## 参考链接

- [Build A Python Speech Assistant App](https://www.youtube.com/watch?v=x8xjj6cR9Nc)
- [Rasa Masterclass: Developing Contextual AI assistants with Rasa tools](https://www.youtube.com/playlist?list=PL75e0qA87dlHQny7z43NduZHPo6qd-cRc)
- 自然语言处理实战(聊天机器人技术原理与应用)
