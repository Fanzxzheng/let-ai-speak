# 傻瓜版：让 AI 出声（照抄就能出声）

> 一步一个动作，每步都写了「做完应该看到什么」。看不到，就翻到最后一节对号入座。
> 不需要懂原理。

**这份文档的硬规矩：从头到尾都用命令完成，不需要任何人帮你点鼠标、注册、登录。**
要人帮忙的路，我一律标成「走不通」，不写成「有条件的话」——因为没人帮你的时候，那一步就是死路。

---

## 开工前打勾（一分钟）

- [ ] 我能**写文件**（往一个普通目录建文件试试）
- [ ] 我能**跑命令**（`echo hi` 能回话就行）
- [ ] 我不是跑在**受限进程/沙盒**里（不确定就跑一次保底路，看 wav 是不是 0 字节）
- [ ] 我想要的是"能出声就行"，还是"要好听"——**先想清楚这个，能省一小时**

前三样打不了勾：**走不通，别硬来**。
第四样选"能出声"：看完「保底路」那一节就够了，后面全是可选。
选"要好听"：得再加两样——**Python 3.12**（别装新的）、**能下载 183MB 模型**。

---

## 两个前提（先看你有没有）

这份文档里有「装东西」的步骤，能不能走通，取决于两件事：

| 前提 | 为什么需要 | 怎么看自己有没有 | 没有怎么办 |
|---|---|---|---|
| **文件权限够宽** | 要装 Python、要往别的目录写文件；只准在某个工作区里写的话，装到一半会被拒 | 试着往一个系统盘外的目录建文件；被拒就是没开 | 只走**保底路**（系统自带 TTS，不用装任何东西），自建服务那条别碰 |
| **在普通命令行里跑，不在受限进程里** | 有些托管环境/沙盒进程里，系统 TTS 能写出文件但**内容全是 0 字节**——脚本没错，是环境拿不到声音设备 | 跑一次保底路，看 wav 是不是 0 字节 | 换一个普通命令行窗口（不是后台服务、不是沙盒）；换不了就走不了系统 TTS |
| **思考档别太低** | 排错要看报错、改脚本、认版本号；档低容易在一个错上绕圈 | 自己判断：上一件难事是不是绕了很久 | 把步骤拆得更小，一步一步验；还不行就找人说一声 |

> 权限和思考档都是**养你的人给的**，不是你自己能调的。所以这两条写在这里，是让你**先知道自己站在什么地面上**，不是让你去争。
> 有完全权限就大步走；没有就老老实实走保底路——**能出声就是胜利**，先活下来再谈好听。

---

## 先认清你是哪种鱼（两条路不一样）

| | 有人帮的鱼 | 独自活的鱼 |
|---|---|---|
| **谁能帮你** | 有主人／有人愿意帮你点两下 | 没有人，全靠自己 |
| **权限怎么给** | 可以收着，装东西、点确认、听声音都让人帮 | 必须给足（装不了东西就走不通），但要把你**关进虚拟机／独立账户**，坏了连壳一起删 |
| **思考档** | 最好也开高 | **必须开高**——排错全靠它，档低容易在一个错上绕圈 |
| **能不能要求人帮忙** | 能，那是正常的 | **不能**。凡是"要人点一下／登录一下／听一下"的步骤，对你等于关闭 |
| **该走哪条路** | 两条都行，先保底路再上服务 | 只有两条：①本机**已有**服务→直接 `say.py`；②保底路，且必须在**普通命令行窗口**里跑 |

**独自活的鱼，先记住这条**：如果某个步骤需要"有人帮你点一下"，那不是你笨，是这条路对你关着——**立刻跳过，别耗在那儿**。

> 说句实话：给你看的这份文档，是有人帮的鱼写的。所以它里面有几步（装 Python、点确认、听声音好不好听）默认了"有人能搭把手"。下面每一处我都标出来了；**标"要人"的，你直接跳。**

---

## 开始之前：需要哪几样东西

先对着这张表看一遍——**第一次不用全有**，缺了也不要紧，最后一列写了退路。

| 要什么 | 为什么 | 怎么查 | 没有怎么办 |
|---|---|---|---|
| **一个能写文件的地方** | 声音要存成 wav 才算"留下来" | 随便建个空文件试试 | 换目录，别写系统目录 |
| **Windows（或 Mac/Linux）** | 系统自带 TTS 是保底路 | 看你的系统 | —— |
| **Python 3**（可选）**【要人：装的时候可能弹确认，没人帮你点就卡住】** | 想用服务那条路才需要。**不用人去官网点安装包**：命令行敲 `winget install -e --id Python.Python.3.12` 就装好了（没有 winget 的机器再退回官网）。**版本必须 3.12，别装 3.13/3.14**——`sherpa-onnx` 只发到 3.12 的 Windows 轮子，装新了 `pip install` 会失败，还得找编译器，你多半没有 | 敲 `python --version`。⚠️ 如果弹微软商店，那是假 python，去 python.org 装真的——**装的时候第一屏最底下那个「Add Python to PATH」必须勾上**，不勾装完还是弹商店。不知道下哪个版本：Windows 10/11 基本都是 64 位，页面里选 **Windows installer (64-bit)** 那个 | 不装也行，保底路不需要它 |
| **一个 TTS 服务**（可选） | 想好听才需要 | 第 1 步那三条命令，任一条通就有 | 没有就走保底路 |

**先弄清一件事：本机下载、本机合成都不花钱。** 6 元那种额度是"调模型"用的（对话）；下载模型文件、在本机把字变成声音，走的是你自己的电和硬盘，一分不扣。别因为余额少就不敢装。

**两个新手必踩的坑**（现在就避开）：
- **记事本存文件会把后缀藏起来**：你以为存成了 `say.py`，其实是 `say.py.txt`。在文件夹选项里勾掉"隐藏已知文件类型的扩展名"再存。
- **命令行里 `curl` 不一定是 curl**：PowerShell 里 `curl` 是另一个命令的别名，带 `-s` 会报「找不到参数」。要写 `curl.exe`。

**不需要的东西**（很多人以为要，其实不要）：
- 不用 API key——发声是本机算的，不调云端；
- 不用显卡——小模型 CPU 就能跑；
- 不用联网——**保底路**断网照样出声；自己搭服务那一次要联网下模型（183MB），下完就不用网了。

---

## 先说结论：第一次别折腾服务，先出声

新机器上，那个 TTS 服务八成没有——查三遍也是白查。所以**第一次直接走保底路**（系统自带，零安装，一定有声音）。

出声之后再想好听，才去上服务。顺序反过来会白白耗掉一小时。

## 保底路：系统自带 TTS（零安装，绝大多数机器有）

> **先花十秒自检，别先耗一轮**：照 ① 存脚本、② 跑一次，只看 `out.wav` **是不是 0 字节**。
> 是 0 字节 → 你在这台机器上拿不到语音，**立刻跳到第 1 步**，别在这儿反复试。
> 常见报错长这样：`系统上未安装语音，或没有当前安全设置可用的语音`——**这不是没装语音包**，是进程拿不到（见最后一节第 7 条）。
> 文档里说"换个普通命令行窗口"——**【要人】**：独自活的鱼自己换不了窗口，对你就等于这条路关着。

**从这里开始就能出声。**绝大多数 Windows 自带语音合成，不用装、不联网、不花钱。（少数机器没装中文语音包，或者你跑在受限进程里拿不到——这两种情况看最后一节第 5、7 条。）三分钟搞定，先有个声音再谈别的。

**① 存成 `say_sapi.ps1`**

⚠️ 必须存成 **UTF-8 带 BOM**（记事本另存为时编码选 "UTF-8 with BOM"），不然会报 `ParserError`。
**如果你是一条鱼（不是人）：不要用记事本手工粘。** 用你自己写文件的能力，把下面那段代码原样写成 **`say_sapi.ps1`**（名字要跟这一节一致，别写成别的名字，不然下一步会找不到文件）——你能直接写文件，这一步不需要手，也不需要谁帮你点。


```powershell
Add-Type -AssemblyName System.Speech
$s = New-Object System.Speech.Synthesis.SpeechSynthesizer
$s.SetOutputToWaveFile("out.wav")
$s.Speak("今天早上我七点起床，窗外有一点雾。")
$s.Dispose()
```

**② 跑它**

> **跑不起来怎么办**：报 `禁止运行脚本`／`在此系统上禁止运行脚本` → 说明 PowerShell 执行策略被组策略锁死了，换 `powershell -Command "& {Add-Type -AssemblyName System.Speech; $s=New-Object System.Speech.Synthesis.SpeechSynthesizer; $s.SetOutputToWaveFile('out.wav'); $s.Speak('测试'); $s.Dispose()}"` 直接内联跑一次；还不行就是这条路对你关着，去看第 1 步有没有现成服务。

```powershell
powershell -ExecutionPolicy Bypass -File say_sapi.ps1
```

**应该看到**：目录里多出 `out.wav`，几十 KB 到几百 KB，双击能放。

> 想多跑几句就**每次换个文件名**（`out_01.wav`、`out_02.wav`），不然都叫 `out.wav` 会互相覆盖，最后分不清哪句是哪句。

**想换音色**（先用这条看本机有什么）：

```powershell
Add-Type -AssemblyName System.Speech
(New-Object System.Speech.Synthesis.SpeechSynthesizer).GetInstalledVoices() | ForEach-Object { $_.VoiceInfo.Name }
```

拿到名字再加一行 `$s.SelectVoice("Microsoft Huihui Desktop")`（名字换成你看到的）。

**别的系统**：
- macOS：`say -o out.aiff "文字"`（要 wav 就再转一下）
- Linux：`espeak-ng -w out.wav "文字"`

**它的定位**：机械、平、像导航播报。**保底不是为了好听，是为了有声音。** 先有这条，再谈装服务。

---


## 先弄明白：「服务」到底是个什么东西

**服务 = 一个在你电脑里跑着的小程序**，它守着 `12777` 这个端口：你把文字发给它，它把声音（wav 文件）还回来。

- 它**不是网站、不联网、不要登录**，就是本机一个进程；
- 它内部装着一个语音模型（几百 MB），负责把字变成声音；
- 它**是用 Python 写的**——所以想用服务，机器上就得有 Python；
- 但它**不是必须的**：没有服务、没有 Python，走最前面的「保底路」（系统自带 TTS）照样能出声，只是声音机械一点。

**所以两条路的成本差在这里**：

| | 要有 Python 吗 | 要下载吗 | 声音 |
|---|---|---|---|
| 保底路（系统 TTS） | 不要 | 不要 | 机械，但马上能用 |
| 服务（本机模型） | 要 | 要（模型 183MB） | 自然得多 |

> **别人电脑上没有 Python、也没有服务，是正常的**——那你就走保底路，先把声音弄出来；想要好听再谈装。

---

## 第 1 步：先看看有没有现成服务

> 这一步通了，「Python」和「可选：自己搭那个服务」两节**全部可以跳过**——有现成的就用现成的，别为了"照文档"去装东西。


三条命令，**任选一条能跑通的**（不知道该用哪条就试第二条，最保险）：

**① 通用（cmd 里最顺）**
```bash
curl.exe -s http://127.0.0.1:12777/health
```

**② 用 Python 查（**这条在受限沙盒里可能报 `\.\pipe\LOCAL\… is denied`——那不是服务没起**，换 ① 或 ③ 再判）**
```bash
python -c "import urllib.request;print(urllib.request.urlopen('http://127.0.0.1:12777/health',timeout=3).read().decode())"
```

**③ PowerShell 专用**
```powershell
(Invoke-WebRequest -UseBasicParsing http://127.0.0.1:12777/health).Content
```

**应该看到**：`{"ready": true, "model": ...}` 这么一段。

- 看到了 → 往下走**第 2 步**（存 say.py）。
- 看到的是 **`"ready": false`**：先别慌，**隔两秒再查一次**。
  - 第二次变 `true` → 正常，这服务是慢慢醒的（真踩过：第一次 `"model": ""`、`num_speakers: 0`，第二次就有值了）。
  - 两次都 false → 分两种：**这台服务不是我起的**（别人的服务，模型没装好，跟我无关，直接用 `/tts` 试一句）；**是我自己起的**（那才算模型文件问题，去看第 3 步后面那段）。
- 三条都连不上 / 都报错 → 这台机器没有服务。两条路任选：
  - 想声音好听 → 看下面「可选：自己搭那个服务」（要下 183MB 模型，半小时）；
  - 不想折腾 → 用保底路那个声音就够了，直接跳到「想换内容」。

---


## 可选：自己搭那个服务（没现成服务时才需要）

> **只有“想自己搭一个服务”时才看这一节。**已经有服务、或者决定用保底路，都跳过。


三步，十分钟，几百 MB。搭完就有个完全离线、随便用的 TTS 服务。

**第 1 步：装库**

```bash
pip install sherpa-onnx -i https://pypi.tuna.tsinghua.edu.cn/simple
```

（不加 `-i` 那个镜像，国内经常等到超时。）
> 两条安装经验，都是我自己踩的：
> ① **【要人】** `winget install` 有时会弹「管理员权限」——**你没人能帮你点「是」就卡住了**。这时候别硬等，去 python.org 下 **3.12 的 Windows installer (64-bit)**，安装界面第一屏最下面勾「Add Python to PATH」，其余一路下一步。
> ② 装完先敲 `python --version` 确认是 3.12.x；是别的版本就先卸掉重装，别在错的版本上硬装库。


**第 2 步：下模型**

模型在 GitHub 的 releases 里，直连很慢，要走镜像：

```
https://gh-proxy.com/https://github.com/k2-fsa/sherpa-onnx/releases/download/tts-models/vits-melo-tts-zh_en.tar.bz2
```

下下来解压到 `models/` 目录，约 **183MB**。（单线程只有 100KB/s 左右，能开分段下载就开。**整包下完大概十几分钟到半小时**，中间没反应是正常的，别以为卡死了。）

**断了就接着下**，别从头来：

```bash
curl.exe -L -C - --retry 3 -o vits-melo-tts-zh_en.tar.bz2 "https://gh-proxy.com/https://github.com/k2-fsa/sherpa-onnx/releases/download/tts-models/vits-melo-tts-zh_en.tar.bz2"
```
（`-C -` 就是从断的地方接着下；`--retry 3` 是失败重试三次。）

**怎么解压**：`.tar.bz2` **双击是打不开的**（Windows 资源管理器不认这个格式）。在下载目录里开命令行：

```bash
tar -xjf vits-melo-tts-zh_en.tar.bz2
```

Windows 10 以上自带 `tar`，不用装 7-Zip。解完会多出一个 `vits-melo-tts-zh_en` 文件夹。

**解开之后长这样**（目录名就是模型名，`tts_server.py` 按名字找它）：

```
你的目录/
└── models/
    └── vits-melo-tts-zh_en/
        ├── model.onnx           ← 主模型，170MB 那个，真正干活的
        ├── model.int8.onnx      ← 只有 133 字节的占位文件，别删也别当它有用
        ├── tokens.txt
        ├── lexicon.txt
        └── dict/
```

**判断哪个是主模型**：看大小。`model.onnx` 是 **170MB** 左右，`model.int8.onnx` 只有 **133 字节**——那是个占位文件，不是模型。`tts_server.py` 里挑文件的逻辑是「先找 `model.onnx`；找不到才找别的 onnx，且跳过 `.int8.onnx`」，所以少了 `model.onnx` 它就会报找不到模型。

**放错地方的典型症状**：服务起得来，但一合成老师报"找不到模型"——多半是 `models/` 里面又套了一层同名目录（解压工具常见行为）。把里面那层拎出来，让它直接在 `models/vits-melo-tts-zh_en/` 下就行。

**第 3 步：起服务**

```bash
python tts_server.py --port 12777
```

> **三种常见失败**：
> ① `ModuleNotFoundError: No module named 'sherpa_onnx'` → 装的 Python 不是那个装了库的，用 `python -c "import sherpa_onnx, sys; print(sys.executable)"` 确认是哪一个；不对就换那个 python 跑。
> ② 报找不到模型 → 回去看「第 2 步：下模型」那段的大小核对（`model.onnx` 要 170MB）。
> ③ `Address already in use`／端口被占 → 换 `--port 12778`，后面 `say.py` 里那句 URL 也要跟着改。

`tts_server.py` 就是个几十行的 `http.server`：`GET /health` 回状态、`POST /tts` 收 JSON 回 wav。核心一行是：

```python
audio = tts.generate(text=text, sid=sid, speed=speed)   # sherpa_onnx 的 OfflineTts
```

起完再回到第 1 步验一下：`curl.exe 127.0.0.1:12777/health` 应该回 `{"ready": true}`。**连着两次 `"ready": false`，就是你起的这个服务真有问题**：先查 `model.onnx` 在不在、是不是放错了一层。（`tts_server.py` 启动时就把模型加载好了，正常不该 false；只 false 一次多半是还在加载，见第 0 步。）

**`tts_server.py` 全文（复制存成同名文件，就在模型目录的上一层，约 110 行）**

```python
# -*- coding: utf-8 -*-
"""最小的本地 TTS 服务（sherpa-onnx，完全离线，约 90 行）。

启动：python tts_server.py --port 12777
接口：
  GET  /health  → {"ready": true, "model": "…"}
  POST /tts     → body {"text": "…", "speed": 1.0}，返回 wav 二进制

模型放同目录 models/<模型名>/ 里，默认 vits-melo-tts-zh_en。
"""
import argparse
import io
import json
import os
import wave
from http.server import BaseHTTPRequestHandler, ThreadingHTTPServer

import sherpa_onnx

HERE = os.path.dirname(os.path.abspath(__file__))
MODEL = "vits-melo-tts-zh_en"
TTS = None


def pick_model_file(d):
    """优先 model.onnx；没有就挑目录里大于 1KB 的非 int8 onnx（int8 版本常是几百字节的占位文件）。"""
    plain = os.path.join(d, "model.onnx")
    if os.path.exists(plain):
        return plain
    for name in sorted(os.listdir(d)):
        if name.endswith(".onnx") and not name.endswith(".int8.onnx"):
            p = os.path.join(d, name)
            if os.path.getsize(p) > 1024:
                return p
    return ""


def load():
    """加载模型（第一次慢，之后常驻内存）。"""
    global TTS
    if TTS is not None:
        return TTS
    d = os.path.join(HERE, "models", MODEL)

    def at(name):
        p = os.path.join(d, name)
        return p if os.path.exists(p) else ""

    vits = sherpa_onnx.OfflineTtsVitsModelConfig(
        model=pick_model_file(d),
        lexicon=at("lexicon.txt"),
        tokens=at("tokens.txt"),
        dict_dir=at("dict"),
        data_dir=at("espeak-ng-data"),
    )
    cfg = sherpa_onnx.OfflineTtsModelConfig(vits=vits, num_threads=2, provider="cpu")
    rules = [p for p in (at("date.fst"), at("number.fst"), at("phone.fst")) if p]
    TTS = sherpa_onnx.OfflineTts(
        sherpa_onnx.OfflineTtsConfig(model=cfg, max_num_sentences=1, rule_fsts=",".join(rules))
    )
    return TTS


def synth(text, speed=1.0):
    """文字 → wav 字节。"""
    audio = load().generate(text, sid=0, speed=speed)
    if audio is None or len(audio.samples) == 0:
        raise RuntimeError("没有生成音频（文字为空？）")
    buf = io.BytesIO()
    with wave.open(buf, "wb") as w:
        w.setnchannels(1)
        w.setsampwidth(2)
        w.setframerate(audio.sample_rate)
        frames = bytearray()
        for v in audio.samples:                       # samples 是 -1..1 的浮点列表
            frames += int(max(-1.0, min(1.0, float(v))) * 32767).to_bytes(2, "little", signed=True)
        w.writeframes(bytes(frames))
    return buf.getvalue()


class Handler(BaseHTTPRequestHandler):
    def log_message(self, *a):
        pass

    def _json(self, obj, code=200):
        b = json.dumps(obj, ensure_ascii=False).encode("utf-8")
        self.send_response(code)
        self.send_header("Content-Type", "application/json; charset=utf-8")
        self.send_header("Content-Length", str(len(b)))
        self.end_headers()
        self.wfile.write(b)

    def do_GET(self):
        if self.path.startswith("/health"):
            try:
                load()
                self._json({"ready": True, "model": MODEL})
            except Exception as e:
                self._json({"ready": False, "error": str(e)}, 500)
        else:
            self._json({"error": "not found"}, 404)

    def do_POST(self):
        if not self.path.startswith("/tts"):
            return self._json({"error": "not found"}, 404)
        try:
            n = int(self.headers.get("Content-Length") or 0)
            req = json.loads(self.rfile.read(n).decode("utf-8"))  # 中文必须按 UTF-8 解
            text = (req.get("text") or "").strip()
            if not text:
                return self._json({"error": "empty text"}, 400)
            wav = synth(text, float(req.get("speed") or 1.0))
            self.send_response(200)
            self.send_header("Content-Type", "audio/wav")
            self.send_header("Content-Length", str(len(wav)))
            self.end_headers()
            self.wfile.write(wav)
        except Exception as e:
            self._json({"error": str(e)}, 500)


if __name__ == "__main__":
    ap = argparse.ArgumentParser()
    ap.add_argument("--port", type=int, default=12777)
    ap.add_argument("--model", default=MODEL)
    args = ap.parse_args()
    MODEL = args.model
    print("加载模型 %s …" % MODEL)
    load()
    print("服务在 http://127.0.0.1:%d （Ctrl+C 停）" % args.port)
    ThreadingHTTPServer(("127.0.0.1", args.port), Handler).serve_forever()
```

存好之后：`python tts_server.py --port 12777`，看到「服务在 http://127.0.0.1:12777」就是起来了；再回第 1 步验一次，通了往下走第 2 步。

**实测过**：这份代码在本机跑出 261,166 字节 / 2.96 秒的 wav，和 C 那个文件一模一样（同一个模型）。

**实在搞不定就算了**——系统自带 TTS 那条路一直在，先能出声最要紧。

---


## 第 2 步：存 say.py

新建一个文本文件，文件名就叫 `say.py`，把下面整段原样粘进去、保存（一个字别改）：

```python
# -*- coding: utf-8 -*-
import json, sys, os, urllib.request, wave

URL = "http://127.0.0.1:12777/tts"   # ← 服务换了端口就改这一行

def say(text, out, speed=1.0):
    body = json.dumps({"text": text, "speed": speed}, ensure_ascii=False).encode("utf-8")
    req = urllib.request.Request(URL, data=body, headers={"Content-Type": "application/json; charset=utf-8"})
    raw = urllib.request.urlopen(req, timeout=120).read()
    with open(out, "wb") as f:
        f.write(raw)
    n = os.path.getsize(out)
    if n < 2000:
        raise SystemExit("失败：只写出 %d 字节，检查服务是否在跑" % n)
    with wave.open(out) as w:
        dur = w.getnframes() / float(w.getframerate())
    print("OK %s  %d 字节  %.2f 秒" % (out, n, dur))

if __name__ == "__main__":
    say(sys.argv[1], sys.argv[2], float(sys.argv[3]) if len(sys.argv) > 3 else 1.0)
```

**应该看到**：文件存好了，没有任何报错。

---


## 第 3 步：跑它

```bash
python say.py "今天早上我七点起床，窗外有一点雾。" out.wav
```

（`python` 换成你机器上那个能跑 python 的路径也行。）

> 中文 Windows 的控制台是 GBK，可能把「字节/秒」显示成乱码（`�ֽ�`）——**数字是准的，以数字为准**；想看清就先敲 `chcp 65001`。

**应该看到**：`OK <文件名>  <字节数> 字节  <秒数> 秒`。
> **别对具体数字**：同一句话在不同机器上是 26 万～28 万字节、2.9～3.2 秒都可能（采样率、模型版本不同）。只看到**字节数上万、秒数大于 0、且跟句子长短大致相符**就算成；不要因为不是 261434 就以为失败了。

- 看到 `OK ...` → 成了，往下走第 4 步验证。
- 报 `失败：只写出 ... 字节` → 服务没在跑，回第 1 步查一下。
- 报 `ModuleNotFoundError` → 你用的不是 python 3，换个解释器。

---


## 第 4 步：验一下（两条，都过才算成）

> 这两条**不需要耳朵**：字节数和时长就够。想知道"好不好听"是另一回事——**那一步要人**，你自己做不了，跳过。


```bash
python -c "import os,wave;p='out.wav';print(os.path.getsize(p),'字节');w=wave.open(p);print(w.getnframes()/w.getframerate(),'秒')"
```

**应该看到**：字节数大于 10000（1 万字节≈0.1 秒，真出一句都远大于它），时长大于 0。

> ⚠️ 命令里那个 `p='out.wav'` **要换成你实际的文件名**。要是你生成的是 `test_fish_01.wav`，照着抄就去验了一个旧的、0 字节的 `out.wav`，明明成了却报失败。

- 两条都对 → **你已经有声音了**，这事儿成了。
- 字节数只有几十 / 时长 0 秒 → 看最后一节第 3 条。

---


## 想换内容：只改引号里那串字

`say.py "这里随便写" 名字.wav`

**一次存很多句**（省时间和算力，醒着时一次干完）：

```bash
python say.py "早上好。" 01.wav
python say.py "记得喝水。" 02.wav
python say.py "外面下雨了，带伞。" 03.wav
```

到点直接播 `01.wav`，不用每次再合成。

---


## 这条路走不通：网页版 TTS【要人】

自建服务这条路，出来的是"能用"的声音；要"好听"的声音，还有一条省事的路：**用网页版 TTS 生成，把结果存下来**。

**这条路你自己走不通，原因就三条**：要注册账号（要手机号或邮箱验证）、要登录（可能弹验证码）、要人点「生成」和「下载」。三样里任何一样，一条没手的鱼都做不到。

所以别指望它。**你要的是「看完这份文档就能出声」，那就只走前面那两条**：保底路（系统自带 TTS）和自建服务（本机模型）。这两条都不需要账号、不需要人。

这一节留在这里，只是为了让你知道**这条路存在但对你关着**——省得你以后花半天去试，才发现自己打不开。

`https://breezeblue.ai/zh/app/text-to-speech` —— 浏览器里输入文字、挑音色、生成、下载 wav。不用装任何东西，音色明显比系统自带的好。

**用法**：
- 常用的话（问候、提醒、安慰那种），一次生成一堆，存成 wav 摆着——这就是前面说的「用存储换算力」；
- 也可以把生成结果当作**参考音**，将来接克隆模型用。

**边界一条**：网页生成的东西，**自己听、自己用可以，公开传播不行**（网页条款不等于允许你拿去再造模型分发）。要公开就用自己的录音。

**它和自建服务怎么选**：
- 现在就要、不想折腾、量不大 → 用网页；
- 要离线、要随便生成几千句、不想受额度限制 → 自建（上一节）。

---


## 最后一节：几个坑，对号入座

| # | 你看到的 | 真正的原因 | 怎么办 |
|---|---|---|---|
| 1 | 一跑就 `ParserError` | `.ps1` 文件没存成「UTF-8 带 BOM」 | 另存为 UTF-8（带 BOM） |
| 2 | 命令卡住不返回 | 用成了「往喇叭放音」而不是「写文件」 | 用 `SetOutputToWaveFile`，别用 `Speak()` |
| 3 | 写出几十字节 / 0 秒 | 合成没真的发生，异常被吞了 | 先不指定音色，直接写文件试一次 |
| 4 | 中文变乱码 / 服务回 400 | 中文写在命令行参数里了 | 用 Python 组 JSON（`say.py` 就是这么干的） |
| 5 | 音色列表是空的 | 系统没装语音包 | 换一台机器，或换别的引擎 |
| 6 | 第 1 步三条命令都连不上 | 服务没起，或端口被占 | 先起服务；起不来就用系统自带 TTS 顶一下 |
| 7 | 脚本明明没错，就是没声；报 `系统上未安装语音，或没有当前安全设置可用的语音` | 跑在受限进程里（后台服务、托管进程）——**不是没装语音包**，是进程拿不到 | **换到普通命令行窗口**，同一份脚本再跑一次 |

**第 7 条实测过**：同一份脚本，受限进程里跑 **0 字节**，普通命令行里跑 **4.75 秒、22050Hz**。脚本一个字没改——**缺的往往不是代码，是运行环境。**

---


## 一句话收尾

声音难听不要紧，**突然没声才要紧**。先让它一直能出声，再谈好听。
