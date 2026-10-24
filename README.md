# 无界笔记 → A4 分块 PDF

把 Notein（无界笔记）导出的长图切成 A4 版面，逐块取舍、逐页微调，然后直接生成可打印的 PDF。

单文件、纯前端、无依赖、无后端。图片只在浏览器内存里处理，不上传任何服务器。

## 下载

**整个工具就是一个 HTML 文件，下载后双击就能用** —— 不用解压、不用安装、不需要联网。

**[→ 下载 A4-PDF.html](https://raw.githubusercontent.com/b13137981929-debug/notein-a4-pdf/main/A4-PDF.html)**（48 KB）

打开链接后如果显示成源码，按 `Ctrl + S` 保存，文件名填 `A4-PDF.html` 即可。也可以走仓库的文件页：[A4-PDF.html](https://github.com/b13137981929-debug/notein-a4-pdf/blob/main/A4-PDF.html)，点右上角 `Raw` 旁边的下载图标。

想要完整项目（含 README、LICENSE）再拿 zip：[Releases](https://github.com/b13137981929-debug/notein-a4-pdf/releases/latest) 里的 `Source code (zip)`，或仓库右上角绿色 `Code` → `Download ZIP`。

## 它解决什么问题

无界笔记是无限画布，导出的长图动辄几千像素高、比例任意，直接打印会出现两种问题：一是按 A4 缩放后字小得看不清，二是一页塞不下、被打印机硬切成两半，切口正好压在内容上。

这个工具的思路是先把图切成若干「大致接近 A4 比例」的块，让你决定哪些块要、哪些块不要，再按 A4（或 A3/A5）逐页排版，最后生成真正的 PDF —— 而不是靠浏览器打印对话框去猜页边距。

## 主要功能

**两种切法**

- **手动切割**：拖动画布上的绿线决定格子边界；橙色虚线是按当前版面算出的 A4 比例提示，靠近时自动吸附。直接点某个格子就能切换这一页要不要，被点掉的格子压暗并打 ×。
- **A4 取景框**：每加一个框就多一页，框内内容原样搬到 A4 上，框外压暗切掉。框可拖动、拖右下角改大小、滚轮缩放，能加任意多个，各自独立。

**逐页微调**

点缩略图进入单页设置，可以单独换纸张（跟随 A4 / A4 / A3 / A5 / 按内容比例）、单独换方向、单独调缩放（30%–300%），也可以「应用到全部」。超出页面的部分会被裁掉，界面会提示。

**页面顺序**

缩略图下面有 `←` `→` 调序、`×` 删页；另有「恢复全部」「整体反序」两个批量操作。适合那种长图分块后需要重新组织阅读顺序的场景。

**导出**

- 纸张方向：A4 纵向 / 横向，页边距按毫米输入
- 页面缩放：统一比例（所有块同一倍率，保留相对大小）或每页适应（每页各自撑满）
- 画质：标准 0.82 / 高清 0.92 / 最高 0.98（JPEG 质量）
- 右下角自动打「当前页/总页数」，字号 7pt、灰色

**概略图**

预览图上亮的部分会被采用，暗的部分被切掉，每块左上角标页码；块的长宽比与当前版面 A4 比例偏差超过 3% 时描边变橙色，提示这一页可能留白或裁切。

## 使用

1. 下载 `A4-PDF.html`，双击用 Chrome / Edge 打开（不需要装任何东西，也不需要联网）。
2. 在 Notein 里导出「图片 / 长图」，拖进页面。建议宽度不低于 1500px，打印才够清晰。
3. 选模式、调版面，点「生成 PDF」，文件会以 `<原图名>-A4.pdf` 下载。

打印时务必选「实际大小 / 100%」，不要用「适合页面」，否则页边距会被二次缩放。

## 实现要点

整个 PDF 是手写拼出来的，没有引入 pdf-lib / jsPDF 之类的库：

- 单位换算：`1mm = 72/25.4 pt`，A4 即 `595.28 × 841.89 pt`。
- 每块先用离屏 canvas 按原始像素裁切，再 `toBlob('image/jpeg', quality)` 编码。
- PDF 1.4 结构：catalog → pages → 每页 `Page` / `Contents` / `Image` 三个对象，图片以 `/DCTDecode`（JPEG 原始字节）直接嵌入，不重编码；字体只嵌一个 Helvetica 的 Type1 引用，仅用于页码。
- 单页内容流：

  ```
  q
  0 0 <pageW> <pageH> re W n
  <w> 0 0 <h> <x> <y> cm
  /Im0 Do
  Q
  ```

  先按纸张裁剪，再平移缩放到图片实际位置；页码另起一段 `BT ... Tj ET`。
- xref 表在生成时按字节偏移量手工累加写入，跨页尺寸不一致（比如混用 A3 和 A4）也能正常打开。

因为每页 `MediaBox` 独立，所以一份 PDF 里可以混排纵向 A4、横向 A3 和按内容高度的自定义页。

## 浏览器兼容

依赖 Canvas 2D、`canvas.toBlob`、Pointer Events、`TextEncoder`。Chrome / Edge 90+、Safari 15+、Firefox 90+ 均可。目前主要在 Windows + Chrome 下使用。

## 已知限制

- 一次只能处理一张图；多张需分别导出。PDF 也不能直接导入，要先转成图片。
- 单页内容超出页面会被裁掉，不会自动重排文字（本来就是图片）。
- 导出时每页都要完整编码一次，页数多、画质高时会占用较多内存，处理期间页面会短暂卡顿。
- 页数上限 400 页。
- 页码用 Helvetica，只输出数字，不涉及中文字体嵌入。

## 文件结构

```
notein-a4-pdf/
├── A4-PDF.html     # 全部代码：样式、交互、排版、PDF 生成
├── LICENSE
└── README.md
```

## License

[MIT](LICENSE)

---

## English

A single-file, client-side tool that turns a long note image exported from Notein (an infinite-canvas note app) into a print-ready A4 PDF.

Two cutting modes — a draggable grid with A4-ratio snapping, or free A4 viewfinder frames — plus per-page paper size, orientation and zoom, page reordering, and a hand-written PDF writer that embeds JPEG tiles via `/DCTDecode` with no third-party libraries. Everything runs locally in the browser; no upload, no server.

Open `A4-PDF.html` in Chrome or Edge, drop a long image in, and export.
