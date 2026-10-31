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
