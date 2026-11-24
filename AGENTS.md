# Province Translation — 工作约定

纯资源包模组：没有 Java 代码，只有 `assets/mc-province/lang/*.json`、`fabric.mod.json`、图标。
游戏侧逻辑全在 [Province](https://github.com/KaibuchiShuuki/Province) 本体里。改动前先读 `README.md` 的
「How it works / Writing your own language pack」两节，那里写了 id、键名、回退规则。

## 目录

```
mc-province-translation/
├── AGENTS.md              本文件
├── README.md              英文在前、中文在后（同一文件）
├── RELEASE-BODY.md        GitHub Release 正文的源文件，改完用它 PATCH
├── LICENSE                MIT
├── logo.png               300×300，README 与 Release 用
├── build.gradle           纯 resources 打包，processResources 展开版本号
├── gradle.properties      translation_version=0.2.4.0
├── src/main/resources/
│   ├── fabric.mod.json
│   └── assets/mc-province/
│       ├── icon.png       256×256，进 jar
│       └── lang/          zh_tw fr_fr ru_ru es_es ja_jp ko_kr
├── tools/gen_icon.py      从 Province 的 logo/icon 生成翻译版图标
└── 0.2.4.0/               发布产物：jar + changelog
```

## 硬规则

1. **id 必须是 `province-translation`**。本体用 `FabricLoader.isModLoaded` 判断它，
   换 id 会让「跟随系统」选项和设置里的 ? 按钮都不出现（因为没有翻译包时本体只认这个 id）。
2. **不要放 `zh_cn.json` 或 `en_us.json`**。这两个在本体 jar 里，翻译包再放一份会和 classpath 抢同名资源，
   谁先被读到看加载顺序。
3. **键名照抄本体，一个不许改、只许缺**。缺的键本体回退英文，改名就等于那条永远显示英文。
4. 新语种要先在 Province 本体给 `Language` 加枚举项（`resolve()` 里也要给 base code 映射），
   否则语言选择框里没有它，翻译包写了也没入口。
5. 九个语言文件（本体 zh_cn/en_us + 这里七个）键数必须一致，当前 **293 键**；
   本体加键要同步补这里，漏了就是回退英文。
6. 写中文/日文/韩文文件只能用 `write` 工具或 Python 以 UTF-8 写出；
   **不要用 `Set-Content`**（默认 ANSI，会把非 ASCII 写成 GBK 乱码）。
7. 构建：`Push-Location` 进本目录再 `.\gradlew.bat build --offline -q`，不要把路径拼在别的目录上跑。
8. 版本号三段：跟 Province 的 `0.2.4` 加自己的 `0`，即 `0.2.4.0`；jar 名 `ProvinceTranslation-<版本>.jar`。

## 发布流程

1. 改 `gradle.properties` 的 `translation_version`，构建。
2. 产物复制到 `<版本>/`，同时更新 `<版本>/CHANGELOG-<版本>.md`。
3. `RELEASE-BODY.md` 改好，push 代码，再建/改 Release，正文用 UTF-8 提交（见下）。
4. 附件上传时文件名里的 `+` 要 URL 编码，否则会被当成空格。

## GitHub API 坑

- Release 正文中文变问号：发送时编码坏了，不是页面渲染问题。用 Python
  `json.dumps({"body": text}, ensure_ascii=False)` 写 UTF-8 文件，再
  `curl.exe -X PATCH -H "Content-Type: application/json; charset=utf-8" --data-binary "@body.json"`。
  `Invoke-RestMethod -Body` 传内联 JSON 会 400；pwsh 也不支持 bash 的 `<<<`。
- 正文里直接写纯文件名，GitHub 可能把 `ProvinceTranslation-0.2.4.0.jar` 这种路径渲染成链接；
  想让它安静地显示就套反引号。
- 图片要用绝对地址：`https://raw.githubusercontent.com/KaibuchiShuuki/Province-Translation/main/logo.png`。
  README 里相对路径能显示，Release 页面不行。
- 侧栏仓库排序按最近 push，改不了；想让它别老顶在最前面，就少往这个仓库 push。
