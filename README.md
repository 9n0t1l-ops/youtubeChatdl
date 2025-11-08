# YouTube 直播聊天室回放抓取工具

该脚本用于抓取 YouTube Live 的聊天室回放信息，并将其整理为 CSV 文件，以便离线分析或备份。

## 功能简介

- 自动调用 YouTube Live Chat Replay API，按 continuation 分段抓取回放数据。
- 自动解析普通留言与付费留言（Super Chat），保留关键信息。
- 跳过负时间戳、空消息与控制字符，确保输出数据可直接使用。
- 在抓取结束后去重，保证 CSV 中的记录唯一。

## 依赖安装

```bash
pip install yt-dlp requests
```

脚本默认会尝试读取当前目录下的 `www.youtube.com_cookies.txt`，以便在需要登录信息才能访问聊天室记录时成功拉取数据。你可以使用浏览器插件导出 Cookies 并保存为该文件名。

## 使用方法

```bash
python youtubeChatdl.py https://www.youtube.com/watch?v=<视频ID>
```

执行后，脚本会：

1. 调用 `yt-dlp` 获取视频的时长。
2. 从网页源码中提取 API 参数并获取初始 continuation。
3. 循环请求聊天室回放数据，将结果写入 `chatlog.csv`。
4. 当达到视频总时长、遇到重复 continuation 或 continuation 耗尽时自动结束。

抓取进度与日志会实时输出到终端，同时所有评论会追加到 CSV 文件中。CSV 文件的第一行是标题行，字段顺序如下：

- `time`：评论对应的播放时间（格式为 `H:MM:SS` 或 `M:SS`）
- `user`：留言者昵称
- `comment`：留言文本内容

## 代码中保留的留言字段

在解析回放数据时，脚本保留了以下与留言相关的字段：

- `timestampText` / `videoOffsetTimeMsec`：用于计算播放时间与抓取进度。
- `authorName.simpleText`：留言用户的展示名称，用于输出到 CSV。
- `message.runs[*].text`：留言内容（包括付费留言中的文本）。
- `videoOffsetTimeMsec`：以毫秒为单位的时间偏移量，用于判断是否已经达到视频终点以及避免重复抓取。

其中，`time`、`user`、`comment` 会直接写入 `chatlog.csv`，而 `videoOffsetTimeMsec` 会在内存中用于控制抓取流程和去重逻辑。

## 常见问题

- **提示未找到 `ytInitialData` 或 `continuation`**：通常是因为该直播需要登录才能查看历史消息，请确认已提供有效的 Cookies 文件。
- **抓取速度**：脚本默认每轮请求后暂停 0.08 秒，可根据需要调整 `time.sleep` 的间隔。
- **输出编码**：CSV 以 UTF-8 编码保存，兼容主流数据分析工具。

欢迎根据自身需求对脚本进行扩展，例如增加更多字段或改为输出 JSON 等格式。
