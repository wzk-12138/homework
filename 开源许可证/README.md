# 开源许可证 · Apache Maven 许可证合规检测

上海杉达学院《开源创新思维》第 2 章作业 1：用 Apache Creadur RAT 0.18 扫描 Apache Maven 3.9.9 源码包，产出许可证头的合规检测报告。

## 报告文件

`f25011434-王喆楷-Apache Maven-软件许可测试报告.docx`

正文 5 页，按课程模板排版（宋体五号、首行缩进 2 字符、1.25 倍行距），依次为：项目介绍 → 工具选型与安装（含踩坑） → 检测报告 → 逐条分析。

## 检测概要

| 项 | 结果 |
| --- | --- |
| 被检项目 | Apache Maven 3.9.9 官方源码包（ASF 顶级项目，Apache-2.0） |
| 代码规模 | 14 个模块、2099 个文件、约 13 MB，Java 源文件 1025 个 |
| 检测工具 | Apache Creadur RAT 0.18（单 fat-jar，JDK 21 运行，.sha512 校验通过） |
| 通过 | 1478 个（Apache-2.0 1477、MIT 1） |
| 未通过 | 标准文件 548 个（Unknown 547、GPL-2.0 1）；计入归档/二进制/声明共 621 个 |

## 分析结论

- **535 个（97.6%）告警在 `src/test`**：测试用伪仓库、测试 POM、`.sha1/.md5` 校验和、`.snapshot-version`，属测试输入数据，应配置排除规则而非补许可证头。
- **生产代码的真实问题仅 4 个** `package-info.java`：首行是 `// CHECKSTYLE_OFF: RegexpHeader`，包说明由 `maven.mdo` 生成，属有意的既定例外。
- **4 个第三方许可证文本属误报**：`javax.annotation-api-1.3.2.txt` 同时含 CDDL 与 GPL-2.0 表述，RAT 按文本匹配判成 GPL-2.0——工具把「许可证正文」当成了「代码的许可声明」，实际是 CDDL/GPL 双许可，择 CDDL 即可。
- **交叉验证**：Maven 根 `pom.xml` 自带的 `apache-rat-plugin` 排除清单（`src/test/**`、`**/*.svg`、三个许可证文本）与本次 548 个告警高度重合，说明工具输出可信，但只是「待裁定清单」。

## 复现命令

```bash
curl -O https://dlcdn.apache.org/creadur/apache-rat-0.18/apache-rat-0.18-bin.tar.gz
tar -xzf apache-rat-0.18-bin.tar.gz
java -Dfile.encoding=UTF-8 -jar apache-rat-0.18/apache-rat-0.18.jar \
     --counter-max "UNAPPROVED:-1" --output-file rat.txt -- apache-maven-3.9.9
```

注意两点：0.18 里 `-f` 已改义为 `--force`，输出要用 `--output-file`、目录要用 `--` 分隔；默认阈值要求 `UNAPPROVED` 为 0，否则扫描结束会抛异常退出（报告此时已写出），只需报告时用 `--counter-max "UNAPPROVED:-1"`（分隔符是冒号）。
