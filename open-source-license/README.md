# Apache Maven 许可证合规检测

用 **Apache Creadur RAT 0.18** 扫描 Apache Maven 3.9.9 官方源码包，找出缺失许可证头的文件与混入的第三方许可证，并把 548 条原始告警收敛成一份可执行的处置清单。

## 检测对象

| | |
| --- | --- |
| 项目 | Apache Maven 3.9.9（ASF 顶级项目，Apache-2.0 单一许可） |
| 来源 | 官方 `apache-maven-3.9.9-src.zip`（archive.apache.org） |
| 规模 | 14 个模块、2099 个文件、约 13 MB，Java 源文件 1025 个 |

## 检测方法

RAT 是 ASF 官方发布审计工具，逐文件匹配许可证头，输出文件级清单，正适合回答「哪个文件缺头、哪个文件带了别的许可证」。以官方 `-bin` 包解压即用的 fat-jar 运行，扫描对象为 Maven 源码包根目录，结果同时输出文本报告与 XML 结构化报告，便于逐文件统计。

## 检测结果

| 计数器 | 数值 | 说明 |
| --- | --- | --- |
| Approved | 1478 | Apache-2.0 1477、MIT 1 |
| Unapproved | 548 | 标准文件中未通过：Unknown 547、GPL-2.0 1 |
| Archives / Binaries / Notices | 53 / 12 / 8 | 归档、二进制、声明文件，均未通过 |
| Standards / Ignored | 2026 / 4 | 参与判定的文本文件 / 默认排除 |

## 处置结论

原始 548 条告警不等于 548 处违规，逐条核对后分为四类：

1. **535 条（97.6%）在 `src/test`** —— 测试用伪仓库、测试 POM、`.sha1/.md5` 校验和、`.snapshot-version`，是被测试程序读取的输入数据，加头会破坏断言。**处置：配置排除规则**。
2. **4 个 `package-info.java`** —— 首行是 `// CHECKSTYLE_OFF: RegexpHeader`，包说明由 `maven.mdo` 生成，属有意的既定例外。**处置：加入排除清单**。
3. **4 个第三方许可证文本** —— EPL-1.0/2.0、aopalliance、javax.annotation-api，本身就是分发给他人的许可证全文。其中 `javax.annotation-api-1.3.2.txt` 含 CDDL 与 GPL-2.0 两段表述，被判成 GPL-2.0，属**许可证文本误报**；实为 CDDL/GPL 双许可，择 CDDL 即可。**处置：调整白名单，不做代码改动**。
4. **5 个配置/资源文件** —— `m2.conf`、`maven-deps.svg`、示例工程 `App.java`、两份测试说明文本，本无头部惯例。**处置：无需处理**。

**交叉验证**：Maven 根 `pom.xml` 自带的 `apache-rat-plugin` 排除清单（`src/test/**`、`**/*.svg`、三个许可证文本）与上述分类高度重合，说明工具输出可信，但只是「待裁定清单」，必须叠加人工归类。

## 文件说明

| 文件 | 说明 |
| --- | --- |
| `f25011434-王喆楷-Apache Maven-软件许可测试报告.docx` | 课程作业报告，正文 5 页：项目介绍 → 工具选型与安装 → 检测报告 → 逐条分析 |
