# MDSM

**MDSM — Male DeepSeek Mascot** for the DeepSeek Harness Web GUI: a mascot character generated from a supplied photo, shipped inside the plugin, worn by the harness as its background, its brand mark, and its settings surface.

![MDSM character](assets/MDSM.jpg)
![MDSM wallpaper](assets/MDSM-wallpaper.jpg)

## What it does

- **Chat background** — the shipped 16:9 artwork is mounted as a fixed, click-through layer behind the whole GUI. The shell's surface tokens (`--dsw-alias-bg-base`, `--dsw-specific-sidebar-fill`, `--dsw-alias-bg-layer-1/2`) are faded to the configured opacity so the wallpaper actually shows through instead of hiding behind an opaque shell, and the root background is cleared for it.
- **Controls** — opacity of the shell surfaces, wallpaper blur, a dark scrim for readability, and `background-position`, all applied live.
- **Brand mark** — the square MDSM avatar replaces the logo in the sidebar and the conversation hero through the stock `sidebar.brand.mark` and `conversation.hero.brand.mark` slots.
- **Settings section** — an 「MDSM Male DeepSeek Mascot」 page in Settings edits everything, with previews of both artworks, and applies on save.
- **Host half** — serves the artwork and the config from the local DSH web server (`/api/MDSM/...`), so the browser never reaches outside, and stamps the config into the HTML so the GUI comes up already dressed.

## Install

```sh
dsh plugin --profile web add bauerelizabeth07139/MDSM
```

Any spec the plugin manager accepts works — a GitHub shorthand, a full git URL, or a local checkout:

```sh
dsh plugin --profile web add https://github.com/bauerelizabeth07139/MDSM.git
dsh plugin --profile web add C:\path\to\MDSM
```

Then open **Settings → MDSM Male DeepSeek Mascot**. Uninstall with `dsh plugin --profile web remove MDSM`.

## Configuration

The config lives at `$DSH_HOME/MDSM.json` (default `~/.dsh/MDSM.json`) and is edited by the Settings section; it is also reachable over HTTP.

| Field | Default | Meaning |
|---|---|---|
| `wallpaper` | `true` | Wear the MDSM artwork as the GUI background |
| `brand` | `true` | Replace the sidebar and hero logos with the MDSM avatar |
| `surfaceOpacity` | `88` | Shell surface opacity in % — higher means a more opaque shell, a subtler wallpaper (40–100) |
| `blur` | `0` | Gaussian blur applied to the wallpaper, in px (0–24) |
| `scrim` | `35` | Dark scrim over the wallpaper for readable transcript, in % (0–90) |
| `position` | `center` | Wallpaper `background-position`: `center`, `left`, `right`, `top`, `bottom` |

| Route | Method | Purpose |
|---|---|---|
| `/api/MDSM/config` | `GET` / `PUT` | Read / write the config (writes are same-origin only) |
| `/api/MDSM/wallpaper` | `GET` | The 16:9 background artwork |
| `/api/MDSM/mark` | `GET` | The square avatar artwork |

Every value is clamped server-side; unknown keys are dropped.

## Artwork provenance

The character starts from a supplied photo and is produced with the [SenseAudio image generation API](https://docs.senseaudio.cn/api-reference/endpoint/image/sync) in **image-to-image / reference-consistency mode** (`POST /v1/image/sync`, model `doubao-seedream-5-0-260128`, `reference` = the photo, plain white studio backdrop requested) — the model family documented as 参考一致性生成. The shipped assets are then derived from that generation **without further AI editing**:

- **Extraction** — a near-white mask is flood-filled from the image border, so only the background-connected light area becomes transparent; the character's own whites (shirt, highlights) stay opaque. The mask is eroded by 1px and feathered to avoid a light fringe.
- `assets/MDSM-cutout.png` — the raw transparent cutout.
- `assets/MDSM.jpg` — the character on a soft light card (README and Settings preview).
- `assets/MDSM-mark.jpg` — a square head-to-torso crop, centred on the detected face, for the brand marks.
- `assets/MDSM-wallpaper.jpg` — 1536×864, the extracted character at the right third with a soft ground shadow and faint bokeh.

## Development

No build step, no runtime dependencies (React and `@deepseek-ai/cordis` are peers supplied by the harness).

```sh
npm test   # node >= 22: host routes/config/stamp tests + client DOM-stub tests
```

- `lib/index.js` — the host half: config file, three routes, HTML boot stamp.
- `lib/client.js` — the browser half: wallpaper layer, surface fade, brand-mark slots, Settings section.
- `cordis.patch.yml` — the loader row that makes both halves load.

---

## 中文

**MDSM(Male DeepSeek Mascot,DeepSeek 男性吉祥物)** —— DeepSeek Harness 网页端美化插件:由你提供的照片经 AI 生成的角色形象随插件一起分发,由 Harness 当作背景、品牌标识与设置项穿在身上。

### 功能

- **聊天背景**:内置 16:9 素材作为不可点击的全屏背景层;同时把界面的表面色令牌(`--dsw-alias-bg-base`、`--dsw-specific-sidebar-fill`、`--dsw-alias-bg-layer-1/2`)按设定透明度调淡,背景才能真正透出来。
- **可调参数**:界面不透明度、背景模糊、暗色遮罩、壁纸位置,改动即时生效。
- **品牌标识**:通过官方的 `sidebar.brand.mark` 与 `conversation.hero.brand.mark` 插槽,把侧栏与会话标题处的 logo 换成 MDSM 方形头像。
- **设置页**:Settings 里的「MDSM Male DeepSeek Mascot」页面提供素材预览与全部开关,保存即生效。
- **宿主半**:由本地 DSH Web 服务直接提供素材与配置接口(`/api/MDSM/...`),浏览器无需访问外部网络;配置同时被盖进 HTML,页面一打开就是美化后的样子。

### 安装

```sh
dsh plugin --profile web add bauerelizabeth07139/MDSM
```

随后打开 **Settings → MDSM Male DeepSeek Mascot**。卸载:`dsh plugin --profile web remove MDSM`。

### 配置

配置文件为 `$DSH_HOME/MDSM.json`(默认 `~/.dsh/MDSM.json`),字段与取值范围见上方英文表格;服务端会做钳制并丢弃未知字段。

### 素材来源

人物形象由 [SenseAudio 图片生成接口](https://docs.senseaudio.cn/api-reference/endpoint/image/sync)的**图生图/参考一致性模式**生成(`POST /v1/image/sync`,模型 `doubao-seedream-5-0-260128`,`reference` 传入你的照片,提示词要求纯白影棚背景);之后的抠图与合成均为本地像素处理,不再经过 AI:近白掩码从画面边缘洪泛填充,只有与背景连通的浅色区域变透明,人物自身的白色(衬衫、高光)保持不透明,掩码再腐蚀 1px 并羽化以消除白边。三张素材(角色卡、方形头像、16:9 壁纸)全部由此裁切合成。

### 开发

```sh
npm test   # 需要 node >= 22,无任何运行时依赖
```

## License

[MIT](LICENSE)
