# Province Translation

<img src="logo.png" alt="Province Translation" width="120" align="right">

> **English** is below, [中文在 §中文](#中文) — both halves describe the same mod.
> 本文件英文在前、中文在后（[跳到中文](#中文)），两半讲的是同一个模组。

Province Translation gives [Province](https://github.com/KaibuchiShuuki/Province) six more interface languages — 繁體中文, Français, Русский, Español, 日本語 and 한국어 — without touching a single line of game code. Province ships with Simplified Chinese and English only; everything else is text, and text does not belong in the code jar.

Province Translation 给 [Province](https://github.com/KaibuchiShuuki/Province) 补上六种界面语言——繁體中文、Français、Русский、Español、日本語、한국어——一行游戏代码都不用改。Province 本体只带简体中文和英文，其它语言全是文案，文案不该塞进代码包。

---

## How it works

### Text lives on the classpath

Province never keeps strings in Java. Every language is one JSON file at a fixed path inside the jar:

```
assets/mc-province/lang/zh_cn.json
assets/mc-province/lang/en_us.json
```

At runtime `com.mcprovince.core.Lang` asks the class loader for `/assets/mc-province/lang/<code>.json`. Fabric puts every loaded mod's jar on that same class path, so a jar that ships nothing but `assets/mc-province/lang/fr_fr.json` is indistinguishable from the main mod as far as the loader is concerned. That is the entire trick: this mod is a resource pack wearing a mod's clothes.

### A language is available if its file is

The list of candidates is fixed in `com.mcprovince.core.Language`, one enum constant per language with a code: `zh_cn`, `zh_tw`, `en_us`, `fr_fr`, `ru_ru`, `es_es`, `ja_jp`, `ko_kr`.

`Lang.available(language)` then answers a simple question: is this language built in (`zh_cn` / `en_us`), or does its JSON exist on the classpath? Install nothing and six of the eight answer no — which is why the settings screen can offer exactly two choices. Drop this jar in and six more light up.

### Following the game language needs a second signal

A fixed picker is not enough; the mod also wants to follow the game. That path needs two things:

1. **Mapping** — `Language.resolve(gameCode)` folds the game's language code onto one of the eight: `lzh` and `zh_tw` / `zh_hk` / `zh_mo` / `zh_hant` go to Traditional Chinese, other `zh*` to Simplified, `en*` to English, `fr*` / `ru*` / `es*` / `ja*` / `ko*` likewise, anything else to English.
2. **Permission** — with only the base mod installed, "follow the game" is pointless: the game might be in German and there is nothing to follow. So `ProvinceSettings.effectiveLanguage()` clamps the result to Simplified Chinese or English while no translation pack is present, and the picker omits the "Follow the system" entry entirely.

That is what `Lang.translationPackInstalled()` is for. It doesn't scan the classpath; it asks Fabric whether a mod with id `province-translation` is loaded:

```java
FabricLoader.getInstance().isModLoaded("province-translation")
```

One boolean decides two visible behaviours: whether "Follow the system" appears in the list, and whether the little **?** shows up next to the language row.

### Why there is a ? button at all

With only the base mod, a player who wants Korean has no way to discover that a separate download exists. So Province draws a **?** beside the language row whenever `translationPackInstalled()` is false; hovering or clicking it explains, in both languages, that more languages come from Province Translation, and links here. Installing this mod removes the button — there is nothing left to explain.

### Why one file covers every Minecraft version

Nothing in this mod names a Minecraft class, mixes into a Minecraft method, or reads a Minecraft version. It is JSON under a path that Province defined. So the same jar loads on 1.20.1 and on 26.3 alike; the only real requirement is that the Province version understands the path and the keys, which is what the `depends` block states.

### Keys fall back to English, never to raw key names

`Lang.FALLBACK` is English. A key missing from a language file resolves to the English string, so a half-finished translation shows English for the gaps instead of `province.settings.title` leaking onto the screen. That is deliberate: it makes partial translations shippable, and it means a new key added to Province does not break older packs.

---

## Writing your own language pack

Everything below is what this repository does; copy it and change the strings.

### 1. Layout

```
your-pack/
  build.gradle
  gradle.properties
  settings.gradle
  gradlew, gradlew.bat, gradle/wrapper/…
  src/main/resources/fabric.mod.json
  src/main/resources/assets/mc-province/lang/xx_xx.json
```

A plain `java` Gradle project with no dependencies builds this in seconds; there is no Loom, no Minecraft, no mappings.

### 2. `fabric.mod.json`

```json
{
  "schemaVersion": 1,
  "id": "province-translation",
  "version": "0.2.4.0",
  "name": "Province Translation",
  "authors": ["your name"],
  "license": "MIT",
  "environment": "client",
  "depends": { "mc-province": ">=0.2.4" }
}
```

- `environment: client` — Province is client-side only, so is its text.
- `depends` keeps the pack from loading alone, where it would do nothing at all.
- **The id matters.** `province-translation` is the id Province checks for the "Follow the system" entry and the **?** button. Use a different id and your language still appears in the picker and still works when picked by hand — but "follow the game language" will keep clamping to Chinese/English because Province doesn't know a pack is installed.

### 3. The language file

Name it after the code Province expects — one of `zh_tw`, `fr_fr`, `ru_ru`, `es_es`, `ja_jp`, `ko_kr`. Copy `zh_cn.json` from the main mod as the key template (293 keys at the time of writing) and translate the values; leave the keys alone.

Two rules that are easy to get wrong:

- **Do not ship `zh_cn.json` or `en_us.json`.** Both are inside the main mod. Two jars carrying the same path is a coin flip over which one the class loader hands back, and you would be overwriting a language that already works.
- **Do not rename keys or reuse them for other strings.** A key is an API: Province looks it up by name, and a renamed key silently falls back to English.

Placeholders use `{0}`, `{1}`; keep them, and keep their meaning (`{0}` is usually the region name).

### 4. Adding a language Province doesn't know yet

The eight codes are an enum, not a scan. A ninth language needs a new enum constant plus a `resolve()` case in `com.mcprovince.core.Language` — that is a change to the main mod, so send it upstream rather than trying to smuggle it in from a pack. Once the constant exists, a resource-only pack for it works exactly like the six above.

### 5. Build and check

```sh
./gradlew build          # Windows: gradlew.bat build
```

The jar lands in `build/libs/`. Start the game with it in `mods`, open Settings → Interface → Display language, and the new language should be listed. To test "follow the system" without changing your desktop language, switch the game language in Options → Language and reopen the screen.

---

## Using it

1. Drop `ProvinceTranslation-0.2.4.0.jar` into `.minecraft/mods` next to Province. No other dependency, no config file.
2. In game: Province editor panel → **Settings** → **Interface** → **Display language**. The list now has "Follow the system" first, then every language whose text is present.
3. "Follow the system" picks 繁體中文 for a Traditional Chinese game, 简体中文 for Simplified, and one of the six new languages when the game is set to it. A system language with no text falls back to "English (Default)".
4. Remove the jar and Province goes back to its two built-in languages; your saved choice is kept but clamped to Chinese/English.

The **?** button next to the language row disappears while this mod is installed — it only exists to advertise this download.

## Found a translation mistake?

The text is translated by hand, and hands slip. If a line reads wrong, gets cut off, or uses a
term your community doesn't, open an issue at
<https://github.com/KaibuchiShuuki/Province-Translation/issues> — say which language and which
screen, and quote the key if you can find it (`province.settings.language`, for example); a key
turns the fix into a one-line change.

Missing text counts too: if a screen shows English inside another language, that is a key nobody
has translated yet, and reporting it is just as useful.

---

# 中文


## 它是怎么实现的

### 文案放在 classpath 上

Province 本体里不存字符串。每种语言就是 jar 里固定路径下的一个 JSON：

```
assets/mc-province/lang/zh_cn.json
assets/mc-province/lang/en_us.json
```

运行时 `com.mcprovince.core.Lang` 向类加载器要 `/assets/mc-province/lang/<代号>.json`。Fabric 会把所有已加载模组的 jar 都放在同一条 classpath 上，所以一个只装了 `assets/mc-province/lang/fr_fr.json` 的 jar，在加载器眼里和本体没有区别。整个翻译模组就是这一招：穿着模组外衣的资源包。

### 文件在，语言就算有

候选语言写死在 `com.mcprovince.core.Language` 枚举里，每种语言一个常量、一个代号：`zh_cn`、`zh_tw`、`en_us`、`fr_fr`、`ru_ru`、`es_es`、`ja_jp`、`ko_kr`。

`Lang.available(language)` 只回答一个问题：这种语言是本体自带（`zh_cn` / `en_us`），还是 classpath 上真有它的 JSON？什么都不装，八种里有六种答「没有」——设置界面因此只给两个选项。把这个 jar 丢进去，另外六种就亮了。

### 「跟随游戏语言」还需要第二个信号

光有语言列表不够，模组还想跟着游戏走。这条路要两样东西：

1. **归并**——`Language.resolve(gameCode)` 把游戏语言代码折到八种之一：`lzh` 以及 `zh_tw` / `zh_hk` / `zh_mo` / `zh_hant` 归繁体，其余 `zh*` 归简体，`en*` 归英语，`fr*` / `ru*` / `es*` / `ja*` / `ko*` 各自对应，认不出的一律英语。
2. **许可**——只装本体时，「跟随游戏」是空的：游戏可能是德语，而德语没有东西可跟。所以翻译包不在时，`ProvinceSettings.effectiveLanguage()` 把结果夹回简体中文或英文，语言列表里也干脆不出现「跟随系统」这一项。

这就是 `Lang.translationPackInstalled()` 的用途。它不扫 classpath，而是问 Fabric 有没有加载 id 为 `province-translation` 的模组：

```java
FabricLoader.getInstance().isModLoaded("province-translation")
```

一个布尔值决定两处看得见的行为：列表里有没有「跟随系统」，语言行旁边有没有那个小小的 **?**。

### 为什么还要一个问号按钮

只装本体时，想要韩语的玩家没有任何线索知道还有个单独下载。所以只要 `translationPackInstalled()` 为假，Province 就在语言行旁边画一个 **?**；鼠标停上去或者点一下，用中英双语告诉你更多语言来自 Province Translation，并给出下载地址。装上这个模组，按钮自己就消失了——没东西可解释了。

### 为什么一份 jar 能配所有 Minecraft 版本

这个模组里没有任何一行提到 Minecraft 的类、没有 mixin、不读游戏版本，只有 Province 定好路径下的 JSON。所以同一个 jar 在 1.20.1 和 26.3 上都能加载；唯一的要求是那个版本的 Province 认识这个路径和这些键，`depends` 写的就是这件事。

### 缺键回退英文，不会漏出原始键名

`Lang.FALLBACK` 是英文。某种语言里缺了某个键，取到的是英文句子，所以翻了一半的语言包只会在缺口处显示英文，而不是把 `province.settings.title` 这种原始键名甩到屏幕上。这是故意的：半成品也能发布，本体以后加新键也不会把旧语言包弄坏。

## 想自己做一个语言包

下面就是这个仓库的做法，照抄改文案即可。

### 1. 目录

```
your-pack/
  build.gradle
  gradle.properties
  settings.gradle
  gradlew, gradlew.bat, gradle/wrapper/…
  src/main/resources/fabric.mod.json
  src/main/resources/assets/mc-province/lang/xx_xx.json
```

一个不带任何依赖的普通 `java` Gradle 工程，几秒就编完；不用 Loom、不用 Minecraft、不用映射。

### 2. `fabric.mod.json`

```json
{
  "schemaVersion": 1,
  "id": "province-translation",
  "version": "0.2.4.0",
  "name": "Province Translation",
  "authors": ["你的名字"],
  "license": "MIT",
  "environment": "client",
  "depends": { "mc-province": ">=0.2.4" }
}
```

- `environment: client`——Province 本身纯客户端，文案也是。
- `depends` 保证它不会被单独装上去，那样它什么也做不了。
- **id 是有意义的。** `province-translation` 正是 Province 用来判断「跟随系统」和 **?** 按钮的那个 id。换成别的 id，你的语言照样出现在列表里、手动选中也照常生效——但「跟随游戏语言」会一直夹在中文/英文，因为 Province 不知道有翻译包在场。

### 3. 语言文件

文件名要用 Province 认的代号之一：`zh_tw`、`fr_fr`、`ru_ru`、`es_es`、`ja_jp`、`ko_kr`。从本体拷一份 `zh_cn.json` 当键模板（目前 293 个键），只翻值、别动键。

两条容易踩的：

- **不要带 `zh_cn.json` 和 `en_us.json`。** 这两份在本体里。两个 jar 带同一个路径，最后类加载器给你哪一份是掷硬币，而且你覆盖的是本来就能用的语言。
- **不要改键名，也不要把一个键拿去放别的文案。** 键就是接口：Province 按名字查，改名的键会静默退回英文。

占位符是 `{0}`、`{1}`，留着，含义也留着（`{0}` 一般是区划名）。

### 4. 想加 Province 还不认识的语言

八个代号是枚举，不是扫出来的。第九种语言要在 `com.mcprovince.core.Language` 里加一个枚举项和 `resolve()` 的一个分支——那是改本体，所以往上提 PR，别想着从语言包里蒙混进去。枚举项有了之后，只放资源的语言包跟上面六种完全一样。

### 5. 构建与验证

```sh
./gradlew build          # Windows: gradlew.bat build
```

jar 落在 `build/libs/`。丢进 `mods` 启动游戏，打开 设置 → 界面 → 显示语言，新语言应该就在列表里。想验「跟随系统」又不想改系统语言，去游戏「选项 → 语言」切一下游戏语言，再重新打开这个界面即可。

## 怎么用

1. 把 `ProvinceTranslation-0.2.4.0.jar` 和 Province 一起丢进 `.minecraft/mods`。没有别的依赖，也没有配置文件。
2. 游戏里：Province 编辑面板 → **设置** → **界面** → **显示语言**。列表最前面多了「跟随系统」，后面是每一种文案齐全的语言。
3. 「跟随系统」在繁体中文游戏里选繁体，简体游戏里选简体，游戏设成六种新语言之一时选那一种。系统语言没有对应文案时落到「English (Default)」，不会给你一个名字对不上的空选项。
4. 卸掉这个 jar，Province 就回到自带两种语言；你存下的选择还在，只是被夹回中文/英文。

装上之后语言行旁边的 **?** 会消失——它本来就只是为了告诉你这个下载存在。

## 发现翻译错误？

文案是人工翻的，手也会滑。哪一句读着别扭、被截断、或者用了你们那边不这么说的词，去
<https://github.com/KaibuchiShuuki/Province-Translation/issues> 提个 issue：说清是哪种语言、
哪个界面，能带上键名最好（比如 `province.settings.language`），改起来就是一行的事。

漏翻也算：如果某个界面在中文（或别的语言）下冒出了英文，那是还没人翻的键，报上来同样有用。
