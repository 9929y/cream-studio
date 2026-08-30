# background-source.jpg

visionOS 改版用的背景图，Yanice 于 2026-08-27 提供（原件在 `~/Desktop/Cover.jpg`）。

- 原始尺寸 7680 × 4320，1.4 MB
- 实测亮度范围 107–153（中位 112，p95 140）

`index.html` 里**内联的不是这一张**，而是它压到 900px 宽、JPEG quality 62 的版本，只有 12.9 KB
（base64 后约 17 KB）。因为原图本来就是虚化渐变，放大回全屏看不出差别，这样单文件与离线可用都保住了。

要换背景图：把新图按同样方式压一遍再替换 `index.html` 里 `--bg-img` 的 data URI。

```bash
sips -Z 900 --setProperty formatOptions 62 <新图> --out bg900.jpg
```

⚠️ 换图后要重新验对比度。`index.html` 的 `--scrim` 是按这张图**最亮处 153** 反推出来的
（scrim 0.45 + 玻璃 0.14 → 白字 9.09:1、`--ink-2` 5.87:1、`--ink-3` 4.85:1）。
换一张更亮的图，scrim 要跟着加大，否则参数数值会看不清。

⚠️ 2026-08-27 起，桌面上的原件 `~/Desktop/Cover.jpg` 已经不在了。
**这一份是那张原图仅存的副本**，也随 `9929y/cream-studio` 一起推上了 GitHub。删之前先想清楚。
