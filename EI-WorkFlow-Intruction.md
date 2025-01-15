# 工作流
## 关键概念
### 节点
节点是工作流的关键构成，通过连接不同功能的节点，执行工作流的一系列操作。
工作流的核心节点请查看 《节点说明》。
### 变量
变量用于串联工作流内前后节点的输入与输出，实现流程中的复杂处理逻辑，包含系统变量、环境变量和会话变量。详细说明请参考 《变量》。
### 工作流（Workflow）
适用场景：
面向自动化和批处理情景，适合高质量翻译、数据分析、内容生成、电子邮件自动化等应用程序。该类型应用无法对生成的结果进行多轮对话交互。
常见的交互路径：给出指令->生成内容->结束。
## 变量

## 节点说明
### 节点：`Start`

"开始" 节点是每个工作流应用（Chatflow / Workflow）必备的预设节点，为后续工作流节点以及应用的正常流转提供必要的初始信息，例如应用使用者所输入的内容

**属性：**

- `id` (String): ""，
- `name` (String): "开始"，
- `type` (String): "Start"，
- `outputs`(Array):
  -  _(Object):
     -  `name` (String): "query"，
     -  `type`(String): "string"，
     -  `description`(String): "用户输入"，
     -  `required`(Boolean): true，
     -  `source`(String): "system"，
     -  `field_type`(String): "input"，
     -  `reflection`(Boolean): false，
     -  `value`(Object):
        - `type`(String): "literal",
        - `content`(String): "",
        - `hint`(String): "用户输入"
  
### 节点：`LLM`

"LLM"节点调用大语言模型的能力，处理用户输入的信息（自然语言、上传的文件或图片），给出有效的回应信息。

**属性：**

- `id` (String): ""，
- `name` (String): "大模型"，
- `type` (String): "LLM"，
- `inputs`(Array):
  -  _(Object):
     -  `name` (String): "query"，
     -  `description`(String): ""，
     -  `required`(Boolean): false，
     -  `source`(String): "user"，
     -  `reflection`(Boolean): false，
     -  `value`(Object):
        - `type`(String): "ref",
        - `content`(Object):
           - `ref_node_id`(String): "node_start",
           - `ref_var_name`(String): "query",
           - `source`(String): "system"
- `outputs`(Array):
  -  _(Object):
     -  `name` (String): "请输入"，
     -  `type`(String): "string"，
     -  `description`(String): "请输入"，
     -  `required`(Boolean): false，
     -  `source`(String): "user"，
     -  `reflection`(Boolean): false，
     -  `value`(Object):
        - `type`(String): "generated"
- `configs`(Object):
  -  `top_p` (Number): 0.5，
  -  `template_content`(String): "{{query}}"，
  -  `temperature`(Number): 0.5，
  -  `model`(Object): 
     -  `model_name`(String): ""，
     -  `model_type`(String): ""，
     -  `model_deployment_id`(Sting): ""
  - `enable_history`(Boolean): true

### 节点：`Questioner`

"Questioner"节点提供与用户简单交互的能力。

**属性：**

- `id` (String): ""，
- `name` (String): ""，
- `type` (String): "Questioner"，
- `inputs`(Array):
  -  _(Object):
     -  `name` (String): "提问器"，
     -  `description`(String): ""，
     -  `required`(Boolean): false，
     -  `source`(String): "user"，
     -  `reflection`(Boolean): false，
     -  `value`(Object):
        - `type`(String): "ref",
        - `content`(Object):
           - `ref_node_id`(String): "node_start",
           - `ref_var_name`(String): "query",
           - `source`(String): "system"
- `outputs`(Array):
  -  _(Object):
     -  `name` (String): ""，
     -  `cn_name`(String): ""，
     -  `type`(String): "string"，
     -  `description`(String): "用户最近一轮对话输入"，
     -  `required`(Boolean): false，
     -  `source`(String): "system"，
     -  `reflection`(Boolean): false，
     -  `value`(Object):
        - `default`(String): "",
        - `type`(String): "literal",
        - `content`(String): "",
        - `hint`(String): ""
     -  `validator`(Array):
        -  _(Object):
           -  `type`(String): "date_time_format",
           -  `params`(Array): ""
              -  _(String): "%Y-%m-%d %H:%M"
  -  _(Object):
     -  `name` (String): ""，
     -  `cn_name`(String): ""，
     -  `type`(String): "string"，
     -  `description`(String): ""，
     -  `required`(Boolean): false，
     -  `source`(String): "user"，
     -  `reflection`(Boolean): false，
     -  `value`(Object):
        - `default`(String): "",
        - `type`(String): "generated",
        - `content`(String): "",
        - `hint`(String): ""
     -  `validator`(Array):
        -  _(Object):
           -  `type`(String): "date_time_format",
           -  `params`(Array): ""
              -  _(String): "%Y-%m-%d %H:%M" 
- `configs`(Object):
  -  `top_p` (Number): 0.5，
  -  `template_content`(String): "{{query}}"，
  -  `temperature`(Number): 0.5，
  -  `model`(Object): 
     -  `model_name`(String): ""，
     -  `model_type`(String): ""，
     -  `model_deployment_id`(Sting): ""
  - `enable_history`(Boolean): true

## 编排节点
