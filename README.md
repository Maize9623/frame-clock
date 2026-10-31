# 万年历 · 相框版

极简液态玻璃风格的万年历，给电子相框当常驻显示用。单文件产物，不依赖后端，不需要任何 API key。

## 先看效果

`.shots/` 里是用无头 Chrome 在真实分辨率下截的，改样式时可以直接对照。

```bash
npm install
npm run build      # → dist/index.html（单文件，约 52 KB）
npm run dev        # 本地调试，默认 http://localhost:5173
```

`npm run build -- --mode multi` 会输出常规多文件版本到 `dist-multi/`。

## 功能

- 时钟：秒级走时，下方带星期、农历、节气。
- 天气：浏览器定位 → Open-Meteo 免费接口，显示温度、天气、今日高低温、日出日落、体感、风、降水概率；定位被拒时退回设置里的城市名，再失败显示原因，不会白屏。20 分钟自动刷新，回到前台时立刻刷新。
- 万年历：公历 + 农历（含闰月）+ 24 节气 + 干支年 + 生肖，可翻月，回到今天。
- 调休标注：放假格子标“休”并染红，调休上班格子标“班”并染黄。
- 倒计时：内置元旦、春节、清明、劳动节、端午、中秋、国庆，按规则逐年推算，过期自动滚到下一次；设置面板可增删自定义倒计时，支持“每年重复”。
- 白天/黑夜：默认跟随系统 `prefers-color-scheme`，设置里可强制白天或黑夜。
- 相框模式：`?mode=frame` 隐藏设置入口和鼠标痕迹；`?scale=` 调大小；`?vtime=` 调试时钟。

完整参数和操作说明见 [USAGE.html](USAGE.html)。

## 目录

```
index.html                页面骨架
src/main.js               入口：状态、定时器、事件
src/styles/main.css       视觉与响应式布局（纯 CSS，无框架）
src/core/date-utils.js    日期工具
src/core/lunar.js         农历（1900–2100 lunarInfo 表）
src/core/solar-terms.js   24 节气
src/core/holidays.js      节假日索引
src/data/holidays.json    国务院办公厅历年放假与调休数据
src/core/countdown.js     倒计时推算与本地存储
src/core/weather.js       定位 + Open-Meteo
src/core/settings.js      设置与网址参数
src/core/vtime.js         ?vtime= 调试时钟
src/ui/render.js          渲染
src/ui/icons.js           天气图标
```

## 加新的节假日年份

编辑 `src/data/holidays.json`，按已有格式追加，`type` 只有 `off`（放假）和 `work`（调休上班）两种，然后重新 `npm run build`。没有录入的年份，日历底部会显示“××年节假日安排待更新”，不会假装知道。

目前已录入 2024、2025、2026 三年。2026 年安排按国务院办公厅 2025 年 11 月 4 日发布的通知逐条核对；如需正式依据，请以 gov.cn 原文为准。

## 加倒计时

设置面板（齿轮）→“自定义倒计时”；需要常驻的内置节日改 `src/core/countdown.js` 里的 `BUILTIN_FEATURES`。

## 放上相框

1. 相框能直接跑网页：把 `dist/index.html` 拷进去，地址加 `?mode=frame`。
2. 相框只能访问局域网：在电脑上 `python3 -m http.server 8080 --directory dist`，相框打开 `http://<电脑局域网 IP>:8080/?mode=frame`。
3. 定位在 `file://` 下会被浏览器拒绝，给相框固定城市更省事：`?mode=frame&city=杭州`。

设置和倒计时存在相框浏览器自己的 localStorage 里，换设备要重新设置一次。

## 浏览器支持

需要支持 `backdrop-filter`（Chrome 76+/Safari 9+/Edge 79+）和 CSS 容器单位。太老的相框固件若玻璃效果失效，会退化为半透明卡片，布局不受影响。

## 自检记录

日期：2026-10-31。按 `.cursor-task.md` 验证 + 定点修复，未改架构、未加依赖、未 commit。

### 正确性

| 项 | 怎么验 | 结果 |
|----|--------|------|
| 农历 | `/tmp` node 脚本 `import lunarOf`：2026-02-17→正月初一、06-19→五月初五、09-25→八月十五；2027-02-06→正月初一 | 通过，未改 lunarInfo 表 |
| 节气 | 同脚本：`termOn(2026,12,22)==='冬至'`；清明 2026-04-05 | 通过 |
| 节假日 | 对照 `holidays.json` 索引：2026-10-01 休、10-10 班；`hasPlanFor(2027)===false` | 通过 |
| 倒计时 | `computeCountdowns` 钉在 2026-10-31：元旦 62 / 春节→2027-02-06 共 98 天等七项 | 通过 |
| 跨天翻转 | `npm run build` 后 `./bin/vtime-check.sh '2026-10-31T23:59:53+08:00'` → `.shots/auto/rollover.png` | 通过：11 月 1 日，倒计时各 -1 |

### 响应式

`./bin/shots.sh '?city=Hangzhou'` → `.shots/auto/{1920x1200,1280x800,1280x720,1024x768,800x1280,768x1024}.png`。

- 横屏六图：无横向滚动，文字未见溢出裁切。
- **竖屏问题（已修）**：768×1024 倒计时曾被 `.cd-list { min-height:0; overflow:hidden }` + `.glass { overflow:hidden }` + `html/body { height:100% }` 压进视口，后三项不可见且无法滚到。
- **修复**（`src/styles/main.css` 竖屏 media）：`html/body` 改为 `height:auto; min-height:100%`；`.side { min-height:min-content }`；`.countdown` 按内容长高；`.cd-list { overflow:visible; min-height:min-content }`。Puppeteer 复核：`scrollHeight 1190 > clientHeight 1024`，滚到底可见端午/中秋/国庆。

### 边界降级

| 场景 | 怎么验 | 结果 |
|------|--------|------|
| 城市为空 | `?location=city`（无 city） | 天气「未设置城市」，截图 `weather-empty-city.png` |
| 断网/天气失败 | 拦截 `open-meteo.com` | 时钟/日历正常；天气「网络不可用…」（原 `Failed to fetch` 已映射中文） |
| 缺年份 | `?month=2027-02` | 脚注「2027年节假日安排待更新」，`year-2027-missing.png` |
| 坏 localStorage | stub `getItem` 返回非法 JSON、`setItem` 抛错 | 时钟与 7 条倒计时仍渲染，`bad-localstorage.png` |

### 代码清理

删除未使用：`dayDiff`、`LUNAR_MIN_YEAR`、`lunarFull`、`planYearsList`/`basisOf`/`holidayDataYears`、holiday `tint`、`WEEK_HEAD`/`pad`、weather `days`、CSS `.w-temp`、日历重复的 `grid-template-rows`。

### 构建

`npm run build` 通过；`dist/index.html` 单文件约 **52.7 KB**（修前约 52.8 KB，未见膨胀）。
