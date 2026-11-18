# v0.1.6 TXT、静态阅读工作台与DSH安装修复

## 已实现
- Agent prepare_document支持TXT（UTF-8/BOM、UTF-16 BOM、GB18030），本地重排PDF，不依赖Office。网页版支持UTF-8/GB18030 TXT重排，复用100页限制。
- 网页导出HTML包含工作台页眉、页目录、打印、可调分栏；手机上下排列。Agent继续使用同一阅读工作台布局。
- 新增随包paper-margin-ocr Skill、local_ocr_status和ocr_paper_page工具。使用本地Tesseract TSV输出段落坐标，不向云端发送图像；之后仍AI智能分段。MCP共13个工具。

## OCR并非零依赖全集
安装阶段需要Tesseract可执行程序及eng/chi_sim语言包。Python包和ZIP不包含各平台原生二进制。安装后语言包在本地可离线使用。当前机器缺Tesseract，已验证缺依赖提示及模拟TSV坐标解析，未验证真实识别。Android/Termux需要实际提供Tesseract和语言包，且Python核心的PyMuPDF也必须可安装。**当前不能承诺Android16开箱即用OCR。**随包Skill负责检测和安装指引，而不是复用Windows专属MinerU路径。

## DSH disconnected 排查
用户提供环境：Android16，本地DSH 0.2.2-rc.1。
可复现的包侧问题：旧包仅peer声明MCP client，未声明Skill filesystem provider；本机profile对这两个模块均MODULE_NOT_FOUND。旧bundle还绕过内置MCP挂载，加载自定义适配模块。
修复：显式依赖两个0.2.2-rc.1 provider，bundle直接使用内置MCP client；隔离npm安装后两模块解析成功。没有改用户现有profile或重启引擎。
尚不能确认Android的最终连接：还需确认插件目录已真正加入web profile bundles，不只是npm link；两provider可从profile解析；PAPER_MARGIN_PYTHON指向Android实际虚拟环境Python；该环境能import mcp,pymupdf,paper_margin；最后重启引擎并验证tools/list返回13工具。配置合成成功不等于stdio握手成功。Windows隔离依赖测试不替代Android实际握手。

## 安装建议
在长期目录解压；Python虚拟环境中pip install .；DSH通过插件管理安装本目录以解析package依赖，单纯链接且不安装依赖仍可能失败。设置PAPER_MARGIN_PYTHON绝对路径并重启引擎。若缺原生依赖应明确中止，不反复重连掩盖根因。OCR依赖按skills/paper-margin-ocr/SKILL.md配置。不要把Windows路径复制到Android。

## 验证
18项Python/MCP测试通过，含TXT生成、GB18030和模拟OCR坐标；网页TypeScript生产构建通过，浏览器TXT导入通过；两个DSH provider隔离解析通过。未做真实Android引擎握手、真实扫描OCR、像素级视觉冷读。先前功能测试不代表这些缺口已验收。
