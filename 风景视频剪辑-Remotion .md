# 风景视频剪辑 - Remotion 教程

## 项目概述

将 11 个风景视频片段拼接成一个完整风景短片，自动匹配字幕、添加背景音乐，适用于短视频发布、旅拍宣传、风景展示等场景。

## 视频素材清单

|片段序号|源文件|展示字幕|
|-|-|-|
|1|sc1.mp4|黄山（安徽）|
|2|sc2.mp4|张家界（湖南）|
|3|sc3.mp4|九寨沟（四川）|
|4|sc4.mp4|桂林山水（广西）|
|5|sc5.mp4|西湖（浙江杭州）|
|6|sc6.mp4|丽江古城（云南）|
|7|sc7.mp4|布达拉宫（西藏）|
|8|sc8.mp4|长城（北京）|
|9|sc9.mp4|洱海（云南大理）|
|10|sc10.mp4|喀纳斯湖（新疆）|
|11|sc11.mp4|三亚亚龙湾（海南）|

## 核心技术参数

* **分辨率**：1920×1080（1080P 标准）
* **帧率**：30fps
* **单片段时长**：8 秒
* **总时长**：88 秒（11×8 秒）
* **总帧数**：2640 帧
* **输出格式**：MP4（H.264 编码）

## 项目文件结构

```Plain Text
remotion-scenery-edit/
├── src/
│   ├── Root.tsx          # 项目配置、视频参数定义
│   └── SceneryEdit.tsx   # 剪辑核心逻辑、视频+字幕+音频渲染
├── public/
│   ├── videos/           # 存放所有风景视频素材
│   │   ├── sc1.mp4 \~ sc11.mp4
│   └── bgm.m4a           # 背景音乐文件
└── out/
    └── scenery\_video.mp4 # 最终渲染输出视频
```

## 完整配置步骤

### 1\. 安装项目依赖

打开终端，进入项目根目录，执行安装命令：

```bash
cd remotion-scenery-edit
pnpm install
```

### 2\. 放置视频素材

将所有风景视频复制到 `public/videos/` 文件夹，示例命令：

```bash
cp "D:/you/OneDrive/文档/sc\*.mp4" public/videos/
```

### 3\. 配置 Root.tsx（视频总配置）

```tsx
import { Composition } from "remotion";
import { staticFile } from "remotion";
import SceneryEdit from "./SceneryEdit";

// 视频根配置：分辨率、帧率、时长、素材路径
export default function Root() {
  return (
    <Composition
      id="SceneryVideo"
      component={SceneryEdit}
      durationInFrames={2640} // 总帧数 88s\*30fps
      fps={30}
      width={1920}
      height={1080}
      // 默认素材与字幕配置
      defaultProps={{
        videoPaths: \[
          staticFile("videos/sc1.mp4"),
          staticFile("videos/sc2.mp4"),
          staticFile("videos/sc3.mp4"),
          staticFile("videos/sc4.mp4"),
          staticFile("videos/sc5.mp4"),
          staticFile("videos/sc6.mp4"),
          staticFile("videos/sc7.mp4"),
          staticFile("videos/sc8.mp4"),
          staticFile("videos/sc9.mp4"),
          staticFile("videos/sc10.mp4"),
          staticFile("videos/sc11.mp4"),
        ],
        subtitles: \[
          "黄山（安徽）",
          "张家界（湖南）",
          "九寨沟（四川）",
          "桂林山水（广西）",
          "西湖（浙江杭州）",
          "丽江古城（云南）",
          "布达拉宫（西藏）",
          "长城（北京）",
          "洱海（云南大理）",
          "喀纳斯湖（新疆）",
          "三亚亚龙湾（海南）",
        ],
        musicPath: staticFile("bgm.m4a"),
      }}
    />
  );
}
```

### 4\. 核心剪辑组件 SceneryEdit.tsx

```tsx
import {
  AbsoluteFill,
  Audio,
  Video,
  useVideoConfig,
  interpolate,
} from "remotion";
import React from "react";

// 定义组件接收的参数类型
type Props = {
  videoPaths: string\[];
  subtitles: string\[];
  musicPath: string;
};

export default function SceneryEdit({
  videoPaths,
  subtitles,
  musicPath,
}: Props) {
  const { fps, durationInFrames } = useVideoConfig();
  const clipDuration = 8 \* fps; // 每个片段 8 秒

  return (
    <AbsoluteFill style={{ backgroundColor: "#000" }}>
      {/\* 遍历渲染所有视频片段 \*/}
      {videoPaths.map((src, index) => {
        const startTime = index \* clipDuration;
        const endTime = (index + 1) \* clipDuration;

        return (
          <React.Fragment key={index}>
            {/\* 视频片段 \*/}
            <Video
              src={src}
              startFrom={0}
              startFrame={startTime}
              endFrame={endTime}
              style={{
                width: "100%",
                height: "100%",
                objectFit: "cover",
              }}
            />

            {/\* 对应字幕（顶部显示） \*/}
            <AbsoluteFill
              style={{
                justifyContent: "flex-start",
                alignItems: "center",
                paddingTop: 60,
              }}
            >
              <div
                style={{
                  padding: "12px 24px",
                  background: "linear-gradient(to bottom, #00000099, transparent)",
                  borderRadius: 8,
                  fontFamily: "Microsoft YaHei, sans-serif",
                  fontSize: 56,
                  color: "white",
                  textShadow: "0 2px 8px rgba(0,0,0,0.4)",
                  opacity: interpolate(
                    useVideoConfig().frame,
                    \[startTime, startTime + 15, endTime - 15, endTime],
                    \[0, 1, 1, 0]
                  ),
                }}
              >
                {subtitles\[index]}
              </div>
            </AbsoluteFill>
          </React.Fragment>
        );
      })}

      {/\* 背景音乐（全程播放） \*/}
      <Audio src={musicPath} volume={0.7} />
    </AbsoluteFill>
  );
}
```

### 5\. 一键渲染最终视频

```bash
pnpm build
```

## 字幕样式规范

* 位置：屏幕顶部居中
* 字体：Microsoft YaHei（微软雅黑）
* 字号：56px
* 颜色：纯白色
* 效果：淡入淡出 + 文字阴影
* 背景：顶部半透明黑色渐变

## 重要注意事项

1. **背景音乐**：优先使用轻音乐、钢琴、古筝、国风纯音乐，时长建议 ≥ 90 秒
2. **素材统一**：提前统一视频亮度、对比度、色调，提升成片质感
3. **时长调整**：每个片段固定 8 秒，可根据实际需求调整
4. **输出规格**：1080P/30fps，兼容短视频平台

## 输出文件信息

* 输出路径：`C:/Users/you/remotion\\-scenery\\-edit/out/scenery\\\_video\\.mp4`
* 格式：标准 MP4

