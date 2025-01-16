# ModelArts工作流
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
每个节点都是json格式，其属性描述中，括号中的String、Array、Boolean、Object、Number分别对应jsonl文件支持的4种数据类型，用于表明括号前的字段的数据类型，且字段间通过层次关系表示它们间的主从。
每个节点的`"id"`字段的取值必须确保唯一，格式参考"node_1733731697635"，以"node_"开头，后面是13位数字。
在有些场景下，节点中的`"inputs"`和`"outputs"`都可能会包含多个Object对象。
节点中的各字段的实际取值需要根据工作流设置，下面给出的是一些样例。

### 节点：`Start`

"开始" 节点是每个ModelArts工作流应用（Chatflow / Workflow）必备的预设节点，为后续工作流节点以及应用的正常流转提供必要的初始信息，例如应用使用者所输入的内容。
实际应用中有几点需要注意：

- 在有些场景下`"inputs"`和`"outputs"`中都可能会包含多个Object对象

**属性：**

- `"id"` (String): ""，
- `"name"` (String): "开始"，
- `"type"` (String): "Start"，
- `"outputs"`(Array):
  -  `_`(Object):
     -  `"name"` (String): "query"，
     -  `"type"`(String): "string"，
     -  `"description"`(String): "用户输入"，
     -  `"required"`(Boolean): true，
     -  `"source"`(String): "system"，
     -  `"field_type"`(String): "input"，
     -  `"reflection"`(Boolean): false，
     -  `"value"`(Object):
        - `"type"`(String): "literal",
        - `"content"`(String): "",
        - `"hint"`(String): "用户输入"
  
### 节点：`LLM`

"LLM"节点调用大语言模型的能力，处理用户输入的信息（自然语言、上传的文件或图片），给出有效的回应信息。

**属性：**

- `"id"` (String): ""，
- `"name"` (String): "大模型"，
- `"type"` (String): "LLM"，
- `"inputs"`(Array):
  -  `_`(Object):
     -  `"name"` (String): "query"，
     -  `"description"`(String): ""，
     -  `"required"`(Boolean): false，
     -  `"source"`(String): "user"，
     -  `"reflection"`(Boolean): false，
     -  `"value"`(Object):
        - `"type"`(String): "ref",
        - `"content"`(Object):
           - `"ref_node_id"`(String): "node_start",
           - `"ref_var_name"`(String): "query",
           - `"source"`(String): "system"
- `"outputs"`(Array):
  -  `_`(Object):
     -  `"name"` (String): "请输入"，
     -  `"type"`(String): "string"，
     -  `"description"`(String): "请输入"，
     -  `"required"`(Boolean): false，
     -  `"source"`(String): "user"，
     -  `"reflection"`(Boolean): false，
     -  `"value"`(Object):
        - `"type"`(String): "generated"
- `configs`(Object):
  -  `"top_p"` (Number): 0.5，
  -  `"template_content"`(String): "{{query}}"，
  -  `"temperature"`(Number): 0.5，
  -  `"model"`(Object): 
     -  `"model_name"`(String): ""，
     -  `"model_type"`(String): ""，
     -  `"model_deployment_id"`(String): ""
  - `"enable_history"`(Boolean): true

### 节点：`Questioner`

"Questioner"节点提供与用户简单交互的能力。

**属性：**

- `"id"` (String): ""，
- `"name"` (String): ""，
- `"type"` (String): "Questioner"，
- `"inputs"`(Array):
  -  `_`(Object):
     -  `"name"` (String): "提问器"，
     -  `"description"`(String): ""，
     -  `"required"`(Boolean): false，
     -  `"source"`(String): "user"，
     -  `"reflection"`(Boolean): false，
     -  `"value"`(Object):
        - `"type"`(String): "ref",
        - `"content"`(Object):
           - `"ref_node_id"`(String): "node_start",
           - `"ref_var_name"`(String): "query",
           - `"source"`(String): "system"
- `"outputs"`(Array):
  -  `_`(Object):
     -  `"name"` (String): ""，
     -  `"cn_name"`(String): ""，
     -  `"type"`(String): "string"，
     -  `"description"`(String): "用户最近一轮对话输入"，
     -  `"required"`(Boolean): false，
     -  `"source"`(String): "system"，
     -  `"reflection"`(Boolean): false，
     -  `"value"`(Object):
        - `"default"`(String): "",
        - `"type"`(String): "literal",
        - `"content"`(String): "",
        - `"hint"`(String): ""
     -  `"validator"`(Array):
        -  `_`(Object):
           -  `"type"`(String): "date_time_format",
           -  `"params"`(Array): ""
              -  `_`(String): "%Y-%m-%d %H:%M"
  -  `_`(Object):
     -  `"name"` (String): ""，
     -  `"cn_name"`(String): ""，
     -  `"type"`(String): "string"，
     -  `"description"`(String): ""，
     -  `"required"`(Boolean): false，
     -  `"source"`(String): "user"，
     -  `"reflection"`(Boolean): false，
     -  `"value"`(Object):
        - `"default"`(String): "",
        - `"type"`(String): "generated",
        - `"content"`(String): "",
        - `"hint"`(String): ""
     -  `validator`(Array):
        -  `_`(Object):
           -  `"type"`(String): "date_time_format",
           -  `"params"`(Array): 
              -  `_`(String): "%Y-%m-%d %H:%M" 
- `configs`(Object):
  -  `"top_p"` (Number): 0.5，
  -  `"extract_fields_from_response"`(Boolean): true,
  -  `"with_chat_history"`(Boolean): true,
  -  `"extra_prompt_for_fields_extraction"`(String): "如果用户发起修改，只改用户修改的变量，不修改的保持原样",
  -  `"temperature"`(Number): 0.2，
  -  `"max_response"`(Number): 3,
  -  `"question_content"`(String): "",
  -  `"model"`(Object): 
     -  `"model_name"`(String): ""，
     -  `"model_type"`(String): ""，
     -  `"model_deployment_id"`(Sting): ""

### 节点：`Code`

"Code"节点支持运行 Python 代码以在工作流程中执行数据转换。它可以简化你的工作流程，适用于Arithmetic、JSON transform、文本处理等情景。
该节点极大地增强了开发人员的灵活性，使他们能够在工作流程中嵌入自定义的 Python 脚本，并以预设节点无法达到的方式操作变量。

**属性：**

- `"id"` (String): ""，
- `"name"` (String): "代码"，
- `"type"` (String): "Code"，
- `"inputs"`(Array):
  -  `_`(Object):
     -  `"name"` (String): ""，
     -  `"type"`(String): "string"，
     -  `"description"`(String): ""，
     -  `"required"`(Boolean): false，
     -  `"source"`(String): "user"，
     -  `"reflection"`(Boolean): false，
     -  `"value"`(Object):
        - `"type"`(String): "generated"
- `"outputs"`(Array):
  -  `_`(Object):
     -  `"name"` (String): ""，
     -  `"type"`(String): "string"，
     -  `"description"`(String): ""，
     -  `"required"`(Boolean): false，
     -  `"source"`(String): "user"，
     -  `"reflection"`(Boolean): false，
     -  `"value"`(Object):
        - `"type"`(String): "generated"
- `"configs"`(Object):
  -  `"code"` (String): ""

### 节点：`Branch`

"Branch"节点用于设置ModelArts工作流中的分支流程，根据 If/else/elif 条件将 Chatflow / Workflow 流程拆分成多个分支。

**属性：**

- `"id"` (String): ""，
- `"name"` (String): "判断"，
- `"type"` (String): "Branch"，
- `"branches"`(Array):
   - `_`(Object):
     - `"id"`(String): "if",
     - `"configs"`(Object):
       - `"logic"`(String): "and",
       - `"conditions"`(Array):
         - `_`(Object):
           - `"operator"`(String): "eq",
           - `"left"`(Object):
             - `"name"`(String): "",
             - `"description"`: "",
             - `"required"`(Boolean): false,
             - `"source"`(String): "user",
             - `"value"`(Object):
               - `"type"`(String): "ref",
               - `"content"`(Object):
                 - `"ref_node_id"`(String): "",
                 - `"ref_var_name"`(String): "continue",
                 - `"source"`(String): "user"
           - `"right"`(Object):
             - `"name"`(String): "",
             - `"description"`: "",
             - `"required"`(Boolean): false,
             - `"source"`(String): "user",
             - `"value"`(Object):
               - `"type"`(String): "literal",
               - `"content"`(String): "Y",
               - `"hint"`(String): ""
   - _(Object):
     - `"id"`(String): "default"

### 节点：`IntentDetection`

"IntentDetection"节点通过定义分类描述，问题分类器能够根据用户输入，使用 LLM 推理与之相匹配的分类并输出分类结果，向下游节点提供更加精确的信息。

**属性：**

- `"id"` (String): ""，
- `"name"` (String): "意图分类"，
- `"type"` (String): "IntentDetection"，
- `"inputs"`(Array):
  - `_`(Object):
    - `"name"`(String): "input",
    - `"description"`(String): "",
    - `"required"`(Boolean): false,
    - `"source"`(String): "user",
    - `"reflection"`(Boolean): false
    - `"value"`(Object):
      - `"type"`(String): "ref",
      - `"content"`(Object):
        - `"ref_node_id"`(String): "node_start",
        - `"ref_var_name"`(String): "query",
        - `"source"`(String): "system"
- `"outputs"`(Array):
  - `_`(Object):
    - `"name"`(String): "result",
    - `"description"`(String): "输出的分类分支名",
    - `"required"`(Boolean): false,
    - `"source"`(String): "system",
    - `"reflection"`(Boolean): false
    - `"value"`(Object):
      - `"type"`(String): "generated",
      - `"content"`(String): "",
      - `"hint"`(String): ""
- `"configs"`(Object):
  - `"llm"`(Object):
    - `"temperature"`(Number): 0.5,
    - `"top_p"`(Number): 0.5,
    - `"model"`(Object):
      -  `"model_name"`(String): ""，
      -  `"model_type"`(String): ""，
      -  `"model_deployment_id"`(Sting): ""
  - `"prompt"`(String): ""
- `"branches"`(Array):
  - `_`(Object):
    - `"id"`(String): "branch_1",
    - `"configs"`(Object):
      - `"category"`(String): ""
  - `_`(Object):
    - `"id"`(String): "branch_2",
    - `"configs"`(Object):
      - `"category"`(String): ""

### 节点：`Message`

"Message"节点支持返回ModelArts工作流执行过程中间结果。

**属性：**

- `"id"` (String): ""，
- `"name"` (String): "消息"，
- `"type"` (String): "Message"，
- `"inputs"`(Array):
  -  `_`(Object):
     -  `"name"` (String): "input"，
     -  `"description"`(String): ""，
     -  `"required"`(Boolean): true，
     -  `"source"`(String): "user"，
     -  `"reflection"`(Boolean): false，
     -  `"value"`(Object):
        - `"type"`(String): "ref",
        - `"content"`(Object):
           - `"ref_node_id"`(String): "",
           - `"ref_var_name"`(String): "",
           - `"source"`(String): "user"
        - `"hint"`(String): ""
- `"outputs"`(Array):
  -  `_`(Object):
     -  `"name"` (String): "result"，
     -  `"type"`(String): "string"，
     -  `"description"`(String): "消息输出"，
     -  `"required"`(Boolean): false，
     -  `"source"`(String): "system"，
     -  `"reflection"`(Boolean): false，
     -  `"value"`(Object):
        - `"type"`(String): "generated",
        - `"content"`(String): "",
        - `"hint"`(String): ""
- `configs`(Object):
  -  `"template"`(String): ""

### 节点：`Plugin`

"Plugin"节点提供已构造好的插件能力，用于扩展ModelArts工作流的能力，从而实现更复杂的业务场景。

**属性：**

- `"id"` (String): ""，
- `"name"` (String): ""，
- `"type"` (String): "Plugin"，
- `"inputs"`(Array):
  -  `_`(Object):
     -  `"name"` (String): ""，
     -  `"description"`(String): ""，
     -  `"required"`(Boolean): true，
     -  `"source"`(String): "user"，
     -  `"reflection"`(Boolean): false，
     -  `"value"`(Object):
        - `"type"`(String): "ref",
        - `"content"`(Object):
           - `"ref_node_id"`(String): "",
           - `"ref_var_name"`(String): "",
           - `"source"`(String): "user"
  -  `_`(Object):
     -  `"name"` (String): ""，
     -  `"description"`(String): ""，
     -  `"required"`(Boolean): true，
     -  `"source"`(String): "user"，
     -  `"reflection"`(Boolean): false，
     -  `"value"`(Object):
        - `"type"`(String): "ref",
        - `"content"`(Object):
           - `"ref_node_id"`(String): "",
           - `"ref_var_name"`(String): "",
           - `"source"`(String): "user"
- `"outputs"`(Array):
  -  `_`(Object):
     -  `"name"` (String): ""，
     -  `"description"`(String): ""，
     -  `"required"`(Boolean): true，
     -  `"source"`(String): "user"，
     -  `"reflection"`(Boolean): false，
     -  `"value"`(Object):
        - `"type"`(String): "generated"
- `configs`(Object):
  -  `"id"`(String): ""

### 节点：`TaskFlow`

"TaskFlow"节点提供ModelArts工作流任务规划能力。

**属性：**

- `"id"` (String): ""，
- `"name"` (String): ""，
- `"type"` (String): "Plugin"，
- `"inputs"`(Array):
  -  `_`(Object):
     -  `"name"` (String): ""，
     -  `"description"`(String): ""，
     -  `"required"`(Boolean): false，
     -  `"source"`(String): "user"，
     -  `"value"`(Object):
        - `"type"`(String): "ref",
        - `"content"`(Object):
           - `"ref_node_id"`(String): "",
           - `"ref_var_name"`(String): "",
           - `"source"`(String): "user"
  -  `_`(Object):
     -  `"name"` (String): ""，
     -  `"description"`(String): ""，
     -  `"required"`(Boolean): false，
     -  `"source"`(String): "user"，
     -  `"value"`(Object):
        - `"type"`(String): "ref",
        - `"content"`(Object):
           - `"ref_node_id"`(String): "",
           - `"ref_var_name"`(String): "",
           - `"source"`(String): "user"
- `"outputs"`(Array):
  -  `_`(Object):
     -  `"name"` (String): "results"，
     -  `"description"`(String): ""，
     -  `"required"`(Boolean): false，
     -  `"source"`(String): "user"，
     -  `"value"`(Object):
        - `"type"`(String): "generated",
        - `"content"`: null
- `configs`(Object):
  -  `"model"`(String): "",
  -  `"prompt"`(String): "",
  -  `"planner"`(String): "react",
  -  `"plugins"`(Array):
    -  `"id"`(String): "",
    -  `"name"`(String): "",
    -  `"description"`(String): ""

### 节点：`End`

"End"节点定义一个ModelArts工作流程结束的最终输出内容。每一个工作流在完整执行后都需要至少一个结束节点，用于输出完整执行的最终结果。
结束节点为流程终止节点，后面无法再添加其他节点，工作流应用中只有运行到结束节点才会输出执行结果。若流程中出现条件分叉，则需要定义多个结束节点。
结束节点需要声明一个或多个输出变量，声明时可以引用任意上游节点的输出变量。

**属性：**

- `"id"` (String): ""，
- `"name"` (String): "结束"，
- `"type"` (String): "End"，
- `"inputs"`(Array):
  -  `_`(Object):
     -  `"name"` (String): ""，
     -  `"description"`(String): ""，
     -  `"required"`(Boolean): false，
     -  `"source"`(String): "user"，
     -  `"reflection"`(Boolean): false，
     -  `"value"`(Object):
        - `"type"`(String): "ref",
        - `"content"`(Object):
           - `"ref_node_id"`(String): "",
           - `"ref_var_name"`(String): "",
           - `"source"`(String): "user"
- `"outputs"`(Array):
  -  `_`(Object):
     -  `"name"` (String): "response_content"，
     -  `"type"`(String): "string"，
     -  `"description"`(String): "最终输出"，
     -  `"required"`(Boolean): true，
     -  `"source"`(String): "system"，
     -  `"reflection"`(Boolean): false，
     -  `"value"`(Object):
        - `"type"`(String): "generated"
- `configs`(Object):
  -  `"is_stream_out"` (String): "true"，
  -  `"response_template"`(String): "",
  -  `"response_mode"`(String): "directResponse"


## 编排节点
ModelArts工作流内的节点支持串行和并行的连接模式。

### 串行设计
该结构要求节点按照预设顺序依次执行，每个节点需等待前一个节点完成并输出结果后才能开始工作，有助于**确保任务按照逻辑顺序执行。**
例如
