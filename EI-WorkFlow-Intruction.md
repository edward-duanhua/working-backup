# MA工作流
## 关键概念

### 工作流（Workflow）
适用场景：
面向自动化和批处理情景，适合高质量翻译、数据分析、内容生成、电子邮件自动化等应用程序。该类型应用无法对生成的结果进行多轮对话交互。
常见的交互路径：给出指令->生成内容->结束。

### 节点
节点是MA工作流的关键构成，通过连接不同功能的节点，执行MA工作流的一系列操作。
MA工作流的核心节点请查看 《节点说明》。

### 工作流配置文件
用于MA工作流平台导入和导出的无格式jsonl文件，详细信息请查看《MA工作流配置文件》，要求该文件必须完全遵从《MA工作流配置文件》给出的各种约束，否则为无效文件，MA工作流平台无法正常导入。

## 工作流
MA工作流内的节点支持串行和并行的连接模式。

### 串行设计
该结构要求MA工作流节点按照预设顺序依次执行，每个节点需等待前一个节点完成并输出结果后才能开始工作，有助于**确保任务按照逻辑顺序执行。**

例如，在一个采用串行结构设计的“小说生成”AI应用里，用户输入小说风格、节奏和角色后，LLM按照顺序补全小说大纲、小说剧情和结尾；每个节点都基于前一个节点的输出结果展开工作，确保小说的风格一致性。

### 并行设计
该设计模式允许MA工作流多个节点在同一时间内共同执行，前置节点可以同时触发位于并行结构内的多个节点。并行结构内的节点不存在依赖关系，能够同时执行任务，更好地提升**节点的任务执行效率。**

例如，在某个并行设计的翻译工作流应用内，用户输入源文本触发工作流后，位于并行结构内的节点将共同收到前置节点的流转指令，同时开展多语言的翻译任务，缩短任务的处理耗时。

### 节点间连接
在MA工作流的配置jsonl文件中，节点间的连接关系是通过`"edges"`字段来结构化表示，该字段是`Array`类型，其中每一个item都是一个`Object`,表示2个节点的连接关系，item中的`"source"`表示源节点的`"id"`、`"target"`表示目的节点的`"id"`，如果有多条路径的话，还有一个字段`"branch"`,表示实际连接路径。详细定义可参考如下样例：

```python
{
  "edges":[
    {
      "source": "node_1733450845836",
      "target": "node_1733451357022",
      "branch": "if"
    },
    {
      "source": "node_1733451357022",
      "target": "node_end"
    }
  ]
}
```

## 节点说明
对于下面介绍的各类工作流节点，有以下几类公共要求必须遵从，自动生成时不可违背：
- 每个节点都是json格式，其属性描述中，括号中的String、Array、Boolean、Object、Number分别对应jsonl文件支持的4种数据类型，用于表明括号前的字段的数据类型，且字段间通过层次关系表示它们间的主从。
- 每个节点的`"id"`字段的取值必须确保唯一，格式参考"node_1733731697635"，以"node_"开头，后面是13位数字。
- 在有些场景下，节点中的`"inputs"`和`"outputs"`都可能会包含多个Object对象。
- 属性介绍中，每一行后面 `//`表示后面的信息是对该行前面关键字段的详细描述，其中明确给出一些约束，自动生成取值时必须遵从。
- 属性中列举出来的所有关键字都是必须的，不能缺少，否则对应配置文件是无效的，不能够直接导入MA。
- 节点中的各字段的实际取值需要根据MA工作流进行正确设置，下面给出的是一些样例。

### 节点：`Start`

"开始" 节点是每个MA工作流应用（Chatflow / Workflow）必备的预设节点，为后续MA工作流节点以及应用的正常流转提供必要的初始信息，例如应用使用者所输入的内容。
实际应用中有几点需要注意：

- 在有些场景下`"inputs"`和`"outputs"`中都可能包含多个Object对象

**"开始" 节点属性：**

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

**"开始" 节点参考样例**

```python
	{
        "id": "node_start",
        "name": "开始",
        "type": "Start",
        "outputs": [
          {
            "name": "query",
            "type": "string",
            "description": "用户输入",
            "required": true,
            "source": "system",
            "field_type": "input",
            "reflection": false,
            "value": {
              "type": "literal",
              "content": "",
              "hint": "用户输入"
            }
          }
        ]
      }
```
  
### 节点：`LLM`

"LLM"节点调用大语言模型的能力，处理用户输入的信息（自然语言、上传的文件或图片），给出有效的回应信息。

**"LLM"节点属性：**

- `"id"` (String): ""，
- `"name"` (String): "大模型"，
- `"type"` (String): "LLM"，
- `"inputs"`(Array):
  -  `_`(Object):
     -  `"name"` (String): "query"，   // 参数的名称长度必须大于等于1个字符，并且字符只允许为下面三种类型字母（A-Z或a-z）、数字（0-9）、特殊字符：_
     -  `"description"`(String): ""，
     -  `"required"`(Boolean): false，
     -  `"source"`(String): "user"，
     -  `"reflection"`(Boolean): false，
     -  `"value"`(Object):    
        - `"type"`(String): "ref",   // 支持“引用”和“输入”两种类型,前者支持用户选择工作流中已包含的前置节点的输出变量值；后者支持用户自定义取值
        - `"content"`(Object):
           - `"ref_node_id"`(String): "node_start",
           - `"ref_var_name"`(String): "query",
           - `"source"`(String): "system"
- `"outputs"`(Array):
  -  `_`(Object):
     -  `"name"` (String): "请输入"，
     -  `"type"`(String): "string"，   // 输出参数的类型，可选String、Integer、Number、Boolean
     -  `"description"`(String): "请输入"，
     -  `"required"`(Boolean): false，
     -  `"source"`(String): "user"，
     -  `"reflection"`(Boolean): false，
     -  `"value"`(Object):
        - `"type"`(String): "generated"
- `configs`(Object):
  -  `"top_p"` (Number): 0.5，
  -  `"template_content"`(String): "{{query}}"，   // 写提示词时，支持使用{{variable}}格式引用当前节点输入参数中已定义好的参数
  -  `"temperature"`(Number): 0.5，
  -  `"model"`(Object): 
     -  `"model_name"`(String): ""，
     -  `"model_type"`(String): ""，
     -  `"model_deployment_id"`(String): ""
  - `"enable_history"`(Boolean): true

**"LLM"节点参考样例**

```python
	{
        "id": "node_default_llm_chain",
        "name": "大模型",
        "type": "LLM",
        "inputs": [
          {
            "name": "query",
            "description": "",
            "required": false,
            "source": "user",
            "reflection": false,
            "value": {
              "type": "ref",
              "content": {
                "ref_node_id": "node_start",
                "ref_var_name": "query",
                "source": "system"
              }
            }
          }
        ],
        "outputs": [
          {
            "name": "raw_output",
            "type": "string",
            "description": "该节点原始输出",
            "required": false,
            "source": "user",
            "reflection": false,
            "value": {
              "type": "generated"
            }
          }
        ],
        "configs": {
          "top_p": 0.5,
          "template_content": "{{query}}",
          "temperature": 0.5,
          "model": {
            "model_name": "xw-n1-128k-402-部署-121201",
            "model_type": "ei_pangu",
            "model_deployment_id": "6acb6f59-0f29-4a76-aada-431cb2246208"
          },
          "enable_history": true
        }
    }
```

### 节点：`Questioner`

"Questioner"节点提供与用户简单交互的能力。

**"Questioner"节点属性：**

- `"id"` (String): ""，
- `"name"` (String): ""，
- `"type"` (String): "Questioner"，
- `"inputs"`(Array):
  -  `_`(Object):
     -  `"name"` (String): ""，   // 由开发者自定义，可以通过双花括号形式在后续“问题配置”中被参数“问题”引用
     -  `"description"`(String): ""，
     -  `"required"`(Boolean): false，
     -  `"source"`(String): "user"，
     -  `"reflection"`(Boolean): false，
     -  `"value"`(Object):   // 支持“引用”和“输入”两种类型。
        - `"type"`(String): "ref",
        - `"content"`(Object):
           - `"ref_node_id"`(String): "node_start",
           - `"ref_var_name"`(String): "query",
           - `"source"`(String): "system"
- `"outputs"`(Array):
  -  `_`(Object):
     -  `"name"` (String): ""，
     -  `"cn_name"`(String): ""，
     -  `"type"`(String): "string"，   // 输出参数的类型，可选String、Integer、Number、Boolean
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
  -  `"with_chat_history"`(Boolean): true,   // 开启对话历史后，当前工作流的上下文会带入提问器。
  -  `"extra_prompt_for_fields_extraction"`(String): "如果用户发起修改，只改用户修改的变量，不修改的保持原样",
  -  `"temperature"`(Number): 0.2，
  -  `"max_response"`(Number): 3,   // 该参数指在与用户交互过程中，模型能够持续进行对话而不丧失上下文或性能的最大回合数
  -  `"question_content"`(String): "",   // 该参数将在对话框中原样呈现给用户。如未配置此处，将由大模型根据输出参数描述，自动生成包含所有问题关键词的一个问题
  -  `"model"`(Object): 
     -  `"model_name"`(String): ""，
     -  `"model_type"`(String): ""，
     -  `"model_deployment_id"`(Sting): ""


**"Questioner"节点参考样例**

```python
	{
        "id": "node_1734596963300",
        "name": "是否继续券额预估模型建群",
        "type": "Questioner",
        "inputs": [],
        "outputs": [
          {
            "name": "user_response",
            "cn_name": "",
            "type": "string",
            "description": "用户最近一轮对话输入",
            "required": false,
            "source": "system",
            "reflection": false,
            "value": {
              "default": "",
              "type": "literal",
              "content": "",
              "hint": ""
            },
            "validator": []
          },
          {
            "name": "continue",
            "cn_name": "是否继续",
            "type": "string",
            "description": "目标数据的用户输入中是否体现用户是否继续的意图，如果用户表示继续意图（如OK、好的、嗯）或肯定意图（如对、正确）时，该变量值为Y，如果用户表示不再继续、修改建群需求、否定意图（如不对、不正确、不好等）时，该变量值为N；如果目标数据不包含用户输入或不体现是否继续的意图，该变量值置空",
            "required": true,
            "source": "user",
            "reflection": false,
            "value": {
              "default": "",
              "type": "generated",
              "hint": ""
            },
            "validator": []
          }
        ],
        "configs": {
          "top_p": 0.5,
          "extract_fields_from_response": true,
          "with_chat_history": true,
          "temperature": 0.2,
          "max_response": 3,
          "question_content": "即将创建券额预估模型客群，建群原理：\n对于不同运营场使用不同的模型组合以及专家规则圈选高价值人群，然后根据运营人员录入的预算，再结合领券意向、消费意向等模型对高价值人群进行精筛,创建满足业务预算上限的客群，再通过券额成本预估模型预测最优发放金额。 \n是否需要建群？",
          "model": {
            "model_name": "xw-n1-128k-402-部署-121201",
            "model_type": "ei_pangu",
            "model_deployment_id": "6acb6f59-0f29-4a76-aada-431cb2246208"
          }
        }
    }
```

### 节点：`Code`

"Code"节点支持运行 Python 代码以在MA工作流程中执行数据转换。它可以简化你的工作流程，适用于Arithmetic、JSON transform、文本处理等情景。
该节点极大地增强了开发人员的灵活性，使他们能够在MA工作流程中嵌入自定义的 Python 脚本，并以预设节点无法达到的方式操作变量。

**"Code"节点属性：**

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

**"Code"节点参考样例**

```python
	{
        "id": "node_1733477983207",
        "name": "代码",
        "type": "Code",
        "inputs": [],
        "outputs": [
          {
            "name": "key1",
            "type": "string",   // 输出参数的类型，可选String、Integer、Number、Boolean
            "description": "输出",
            "required": false,   // 选择当前输出参数是否必填
            "source": "user",
            "reflection": false,
            "value": {
              "type": "generated"
            }
          }
        ],
        "configs": {
          "code": "def main(args: dict) -> dict:\n    ret = {\n        \"key0\": args.get('input', 'default'),\n        \"key1\": \"hi\"\n    }\n    return ret"
        }
    }
```

### 节点：`Branch`

"Branch"节点用于设置MA工作流中的分支流程，根据 If/else/elif 条件将 Chatflow / Workflow 流程拆分成多个分支。

**"Branch"节点属性：**

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

**"Branch"节点参考样例**

```python
	{
        "id": "node_1734597323899",
        "name": "判断",
        "type": "Branch",
        "branches": [
          {
            "id": "if",
            "configs": {
              "logic": "and",
              "conditions": [
                {\
                  "operator": "eq",   //条件表达式中间部分，当前支持的比较条件有：equal、not equal、contain、not contain
                  "left": {
                    "name": "",   // 条件表达式左边部分，需要选择来自前序节点的输出参数
                    "description": "",
                    "required": false,
                    "source": "user",
                    "value": {
                      "type": "ref",
                      "content": {
                        "ref_node_id": "node_1734596963300",
                        "ref_var_name": "continue",
                        "source": "user"
                      }
                    }
                  },
                  "right": {
                    "name": "",
                    "description": "",
                    "required": false,
                    "source": "user",
                    "value": {
                      "type": "literal",
                      "content": "Y",
                      "hint": ""
                    }
                  }
                }
              ]
            }
          },
          {
            "id": "default"
          }
        ]
    }
```

### 节点：`IntentDetection`

"IntentDetection"节点通过定义分类描述，问题分类器能够根据用户输入，使用 LLM 推理与之相匹配的分类并输出分类结果，向下游节点提供更加精确的信息。

**"IntentDetection"节点属性：**

- `"id"` (String): ""，
- `"name"` (String): "意图分类"，
- `"type"` (String): "IntentDetection"，
- `"inputs"`(Array):
  - `_`(Object):
    - `"name"`(String): "input",   // 参数名称：默认名称input，为固定值，不可编辑
    - `"description"`(String): "",
    - `"required"`(Boolean): false,
    - `"source"`(String): "user",
    - `"reflection"`(Boolean): false
    - `"value"`(Object):
      - `"type"`(String): "ref",   // 支持“引用”和“输入”两种类型,前者支持用户选择工作流中已包含的前置节点的输出变量值；后者支持用户自定义取值
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
  - `"prompt"`(String): ""   // 高级配置项供进阶开发者修改提示词，如果不配置将会使用系统默认值。提示词的撰写可能影响到意图识别节点的准确性。
- `"branches"`(Array):
  - `_`(Object):
    - `"id"`(String): "branch_1",
    - `"configs"`(Object):
      - `"category"`(String): ""   // 在意图输入框中输入意图描述信息，描述信息为针对该类别的描述语句或者关键词，也将作为大模型进行推理和分类的依据。意图数量为2 ~ 5个
  - `_`(Object):
    - `"id"`(String): "branch_2",
    - `"configs"`(Object):
      - `"category"`(String): ""


**"IntentDetection"节点参考样例**

```python
	{
        "id": "node_1733476738596",
        "name": "意图分类",
        "type": "IntentDetection",
        "inputs": [
          {
            "name": "input",
            "description": "",
            "required": false,
            "source": "user",
            "reflection": false,
            "value": {
              "type": "ref",
              "content": {
                "ref_node_id": "node_start",
                "ref_var_name": "query",
                "source": "system"
              }
            }
          }
        ],
        "outputs": [
          {
            "name": "result",
            "type": "string",
            "description": "输出的分类分支名",
            "required": false,
            "source": "system",
            "reflection": false,
            "value": {
              "type": "generated",
              "content": "",
              "hint": ""
            }
          }
        ],
        "configs": {
          "prompt": "你是一个功能分类器，你可以根据用户的请求提问，和相应的功能类别描述，选择正确的功能帮助用户解决问题。\n如果提问涉及公司办公政策或相关知识的查询或解释，选择 办公知识问答 功能；\n如果提问涉及会议室预定、会议室相关资源查询，选择 会议室预定 功能；\n如果提问涉及特定人员的查找、联系信息或身份确认，选择 查人找人 功能；\n如果提问涉及文字或语音内容的翻译需求，无论是语言之间的转换还是文字的解释，选择 翻译 功能；\n如果提问都不涉及以上需求，选择 拒答 功能。",
          "llm": {
            "temperature": 0.5,
            "top_p": 0.5,
            "model": {
              "model_name": "xw-n1-128k-402-部署-121201",
              "model_type": "ei_pangu",
              "model_deployment_id": "6acb6f59-0f29-4a76-aada-431cb2246208"
            }
          }
        },
        "branches": [
          {
            "id": "branch_1",
            "configs": {
              "category": "会议室预订"
            }
          },
          {
            "id": "branch_2",
            "configs": {
              "category": "办公知识问答"
            }
          },
          {
            "id": "branch_3",
            "configs": {
              "category": "翻译"
            }
          },
          {
            "id": "branch_4",
            "configs": {
              "category": "查人找人"
            }
          },
          {
            "id": "branch_5",
            "configs": {
              "category": "当用户提及需要定制化优惠券的金额时，或明确指定使用券额预估模型建群时，选择该分类，例如：发起一个领券的活动、地铁出行（促三方支付）享受优惠券的活动"
            }
          },
          {
            "id": "branch_6",
            "configs": {
              "category": "去医院"
            }
          }
        ]
    }
```

### 节点：`Message`

"Message"节点支持返回执行过程中间结果。

**"Message"节点属性：**

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
  -  `"template"`(String): ""   // 可撰写指定的回复信息，并以{{参数名称}}的形式插入变量。

**"Message"节点参考样例**

```python
{
	"id": "node_1733450716936",
	"name": "消息",
	"type": "Message",
	"inputs": [
		{
			"name": "input",
			"description": "",
			"required": true,
			"source": "user",
			"reflection": false,
			"value": {
				"type": "ref",
				"content": {
					"ref_node_id": "node_1733450861668",
					"ref_var_name": "name",
					"source": "user"
				},
				"hint": ""
			}
		}
	],
	"outputs": [
		{
			"name": "result",
			"type": "string",
			"description": "消息输出",
			"required": false,
			"source": "system",
			"reflection": false,
			"value": {
				"type": "generated",
				"content": "",
				"hint": ""
			}
		}
	],
	"configs": {
		"template": "这个姓名有问题：{{input}}"
	}
}
```

### 节点：`Plugin`

"Plugin"节点提供已构造好的插件能力。

**"Plugin"节点属性：**

- `"id"` (String): ""，
- `"name"` (String): ""，
- `"type"` (String): "Plugin"，
- `"inputs"`(Array):
  -  `_`(Object):
     -  `"name"` (String): ""，   // 从插件元信息中导入，用户无需手动添加
     -  `"description"`(String): ""，
     -  `"required"`(Boolean): true，
     -  `"source"`(String): "user"，
     -  `"reflection"`(Boolean): false，
     -  `"value"`(Object):
        - `"type"`(String): "ref",   // 支持“引用”和“输入”两种类型
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
- `"outputs"`(Array):   // 输出参数所有信息从插件元信息中导入，用户无需手动添加
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

**"Plugin"节点参考样例**

```python
	{
        "id": "node_1733477873270",
        "name": "translate",
        "type": "Plugin",
        "inputs": [
          {
            "name": "text",
            "description": "待翻译的文本，标点符号都是中文标点符号",
            "required": true,
            "source": "user",
            "reflection": false,
            "value": {
              "type": "ref",
              "content": {
                "ref_node_id": "node_1733477117854",
                "ref_var_name": "text",
                "source": "user"
              }
            }
          }
        ],
        "outputs": [
          {
            "name": "errCode",
            "type": "string",
            "description": "errCode",
            "required": true,
            "source": "user",
            "reflection": false,
            "value": {
              "type": "generated"
            }
          },
          {
            "name": "errMessage",
            "type": "string",
            "description": "errMessage",
            "required": true,
            "source": "user",
            "reflection": false,
            "value": {
              "type": "generated"
            }
          },
          {
            "name": "result",
            "type": "string",
            "description": "result",
            "required": true,
            "source": "user",
            "reflection": false,
            "value": {
              "type": "generated"
            }
          }
        ],
        "configs": {
          "id": "5"
        }
    }
```

### 节点：`TaskFlow`

"TaskFlow"节点提供任务规划能力。

**"TaskFlow"节点属性：**

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

**"TaskFlow"节点参考样例**

```python

```

### 节点：`End`

"End"节点定义一个MA工作流程结束的最终输出内容。每一个MA工作流在完整执行后都需要至少一个结束节点，用于输出完整执行的最终结果。
结束节点为流程终止节点，后面无法再添加其他节点，工作流应用中只有运行到结束节点才会输出执行结果。若流程中出现条件分叉，则需要定义多个结束节点。
结束节点需要声明一个或多个输出变量，声明时可以引用任意上游节点的输出变量。

**"End"节点属性：**

- `"id"` (String): ""，
- `"name"` (String): "结束"，
- `"type"` (String): "End"，
- `"inputs"`(Array):
  -  `_`(Object):
     -  `"name"` (String): "result"，
     -  `"description"`(String): ""，
     -  `"required"`(Boolean): false，
     -  `"source"`(String): "user"，
     -  `"reflection"`(Boolean): false，
     -  `"value"`(Object):
        - `"type"`(String): "ref",
        - `"content"`(Object):
           - `"ref_node_id"`(String): "",
           - `"ref_var_name"`(String): "",
           - `"source"`(String): "user"   // 默认取值为"user"
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
  -  `"response_template"`(String): "{{result}}",   // 可撰写指定的回复信息，并以{{参数名称}}的形式插入变量。
  -  `"response_mode"`(String): "directResponse"

**"End"节点参考样例**

```python
	{
        "id": "node_end",
        "name": "结束",
        "type": "End",
        "inputs": [
          {
            "name": "meetion_room",
            "description": "",
            "required": false,
            "source": "user",
            "reflection": false,
            "value": {
              "type": "ref",
              "content": {
                "ref_node_id": "node_1733477335403",
                "ref_var_name": "result",
                "source": "user"
              }
            }
          },
          {
            "name": "office",
            "description": "",
            "required": false,
            "source": "user",
            "reflection": false,
            "value": {
              "type": "ref",
              "content": {
                "ref_node_id": "node_1733477886970",
                "ref_var_name": "result",
                "source": "user"
              }
            }
          },
          {
            "name": "translate",
            "description": "",
            "required": false,
            "source": "user",
            "reflection": false,
            "value": {
              "type": "ref",
              "content": {
                "ref_node_id": "node_1733477873270",
                "ref_var_name": "result",
                "source": "user"
              }
            }
          },
          {
            "name": "find_people",
            "description": "",
            "required": false,
            "source": "user",
            "reflection": false,
            "value": {
              "type": "ref",
              "content": {
                "ref_node_id": "node_1733477983207",
                "ref_var_name": "key1",
                "source": "user"
              }
            }
          },
          {
            "name": "llm",
            "description": "",
            "required": false,
            "source": "user",
            "reflection": false,
            "value": {
              "type": "ref",
              "content": {
                "ref_node_id": "node_default_llm_chain",
                "ref_var_name": "raw_output",
                "source": "user"
              }
            }
          }
        ],
        "outputs": [
          {
            "name": "response_content",
            "type": "string",
            "description": "最终输出",
            "required": true,
            "source": "system",
            "reflection": false,
            "value": {
              "type": "generated"
            }
          }
        ],
        "configs": {
          "is_stream_out": "true",
          "response_template": "{{meetion_room}}{{office}}{{translate}}{{find_people}}{{llm}}",
          "response_mode": "directResponse"
        }
    }
```

## MA工作流配置文件
MA工作流配置文件是**未格式化jsonl文件**，该文件包含**5个必需的关键字**，分别是`"dsl"`、`"metadata"`、`"plugins"`、`"import_type"`、`"sub_workflows"`。一个完整的、可正常导入MA工作流的jsonl文件必须要包括这5个关键字的相关信息，缺一不可。下面逐一详细介绍每个关键字的相关约束。

-  `"dsl"`是一个`Object`类型，其中包括**6个必需的关键字**，分别是`"id"`、`"name"`、`"description"`、`"nodes"`、`"edges"`、`"layouts"`。
  -  `"id"`关键字是一个String类型的**必需字段**，其内容通过4个"-"连接的5个小写字母或数字组合，其中第1个是长度为8的小写字母或数字组合、中间3个都是长度为4的小写字母或数字组合、最后1个是长度为12的小写字母或数字组合，该String类型的取值必须在整个MA的所有工作流中唯一。
  - `"name"`关键字是一个String类型的**必需字段**，其内容表示当前工作流的名称。
  - `"description"`关键字是一个String类型的**必需字段**，其内容表示当前工作流的详细描述。
  - `"nodes"`关键字是一个Array类型的**必需字段**，其中的每1个item都对应上面《节点说明》中介绍的某个工作流节点。
  - `"edges"`关键字是一个Array类型的**必需字段**，其中的每1个item都对应2个工作流节点的连接关系，详细格式参见上面《节点间连接》描述。
  - `"layouts"`关键字是一个Array类型的**必需字段**，其中的每1个item表示`"nodes"`中节点在MA工作流平台画板中的位置，格式如下：
    ```python
      "layouts": {
     	"node_start": {
     		"x": 210,
     		"y": 234.1666717529297
     	},
     	"node_default_llm_chain": {
     		"x": 1970,
     		"y": 1659.5000457763673
     	},
     	"node_1733476811064": {
     		"x": 1530,
     		"y": 2074.8333892822267
     	},
     	"node_end": {
     		"x": 2410,
     		"y": 922.8333587646484
     	}
      }
    ```
- `"metadata"`是一个`Object`类型的**必需字段**，其中包括**21个必需的关键字**，分别是`"id"`、`"name"`、`"description"`、`"code"`、`"avatar"`、`"status"`、`"visibility"`、`"deleted"`、`"dsl_path"`、`"ir_path"`、`"created_at"`、`"updated_at"`、`"created_by"`、`"creator_id"`、`"updated_by"`、`"updater_id"`、`"project_id"`、`"domain_id"`、`"deploy_wf_version"`、`"workflow_type"`。
   - `"id"`是一个`String`类型的**必需字段**，默认值必须与`"dsl"`中的`"id"`的取值完全一致。
   - `"name"`是一个`String`类型的**必需字段**，默认值必须与`"dsl"`中的`"name"`的取值完全一致。
   - `"code"`是一个`String`类型的**必需字段**，其内容只能包含英文字母和下划线，且不能以下划线开头或结尾。
   - `"avatar"`是一个`String`类型的**必需字段**，其内容是工作流图标的BASE64编码信息。
   - `"status"`是一个`String`类型的**必需字段**，默认值为"draft"。 
   - `"visibility"`是一个`String`类型的**必需字段**，默认值为"project"。 
   - `"deleted"`是一个`Number`类型的**必需字段**，默认值为0。 
   - `"dsl_path"`是一个`String`类型的**必需字段**，其内容格式为5个子字符串拼接，分别是"workflow/flow/" 、`"id"`的值 、"/"、`"id"`的值、".json"，例如 "workflow/flow/a69be318-c262-491b-b58a-a3558c21a517/a69be318-c262-491b-b58a-a3558c21a517.json"。
   - `"ir_path"`是一个`String`类型的**必需字段**，其内容格式为5个子字符串拼接，分别是"workflow/ir/" 、`"id"`的值 、"/"、`"id"`的值、".json"，例如 "workflow/ir/a69be318-c262-491b-b58a-a3558c21a517/a69be318-c262-491b-b58a-a3558c21a517.json"。 
   - `"creator_id"`是一个`String`类型的**必需字段**，取值为小写字母和数字组合的、长度为32的字符串，要求唯一性。 
   - `"updater_id"`是一个`String`类型的**必需字段**，取值为小写字母和数字组合的、长度为32的字符串，缺省取值和`"creator_id"`相同。 
   - `"project_id"`是一个`String`类型的**必需字段**，取值为小写字母和数字组合的、长度为32的字符串"。 
   - `"domain_id"`是一个`String`类型的**必需字段**，取值为小写字母和数字组合的、长度为32的字符串"。 
   - `"deploy_wf_version"`是一个`Number`类型的**必需字段**，取值为长度13的数字组合。 
   - `"workflow_type"`是一个`String`类型的**必需字段**，默认值为"chat"，可选取值"task"。
   - 参考示例如下
     
     ```python
	"metadata": {
		"id": "a3bedb198ad787708a434f7c794fb361e8fc",
		"name": "办公助手_from_jiuwen",
		"description": "办公助手_from_jiuwen",
		"avatar": "",
		"status": "draft",
		"deleted": 0,
		"dsl_path": "workflow/flow/a3bedb198ad787708a434f7c794fb361e8fc/a3bedb198ad787708a434f7c794fb361e8fc.json",
		"ir_path": "workflow/ir/a3bedb198ad787708a434f7c794fb361e8fc/a3bedb198ad787708a434f7c794fb361e8fc.json",
		"created_at": 1734613683088,
		"updated_at": 1734613760646,
		"created_by": "lx-1",
		"creator_id": "d5b74c0f6d314137b975007d62622329",
		"updated_by": "lx-1",
		"updater_id": "d5b74c0f6d314137b975007d62622329",
		"project_id": "bc2903d5916b4278a8265f8d7666c0c0",
		"domain_id": "4bb252e66abb40d8b7aa0411da592422",
		"deploy_wf_version": 1734599787545,
		"workflow_type": "chat"
	},
     ```
- `"plugins"`是一个`Array`类型的**必需字段**，如果`"dsl"中的`"nodes"没有plugins类型的节点，则该字段为[],否则其中每个item都是一个`Object`对象，包括**16个必需的关键字**，分别是`"tool_id"`、`"project_id"`、`"tool_display_name"`、`"tool_desc"`、`"icon"`、`"request_info"`、`"auth_info"`、`"input_schema"`、`"output_schema"`、`"is_input_list"`、`"is_output_list"`、`"type"`、`"creator"`、`"creator_id"`、`"created_on"`、`"updated_on"`。注意，`"dsl"`中的`"nodes"`有几个plugins节点，，这里就有对应数目的item，且这里的`"tool_display_name"`必须和`"dsl"`中的`"nodes"`里面的`"Plugin"`类型的节点中的`"name"`一一对应上。
  - `"project_id"`是一个`String`类型的字段，如果`"dsl"中的`"nodes"有plugins类型的节点，则该字段为``**必需字段**，取值为小写字母和数字组合的、长度为32的字符串，缺省取值和`"metadata"`中的`"project_id"`相同。
- `"import_type"`是一个`String`类型的**必需字段**，默认值为`"workflow"`
- `"sub_workflows"`是一个`Array`类型的**必需字段**，默认值为`[]`

下面是一个配置文件的参考格式样例：
```python
{"dsl":{"id":"d14b2c45-a982-4eb2-b12d-8db4874e0bf9","name":"工作流节点全覆盖-01_1","description":"工作流节点全覆盖","nodes":[{"id":"node_start","name":"开始","type":"Start","outputs":[{"name":"query","type":"string","description":"用户输入","required":true,"source":"system","field_type":"input","reflection":false,"value":{"type":"literal","content":"","hint":"用户输入"}}]},{"id":"node_end","name":"结束","type":"End","inputs":[{"name":"result1","type":"string","description":"最终输出","required":false,"source":"user","reflection":false,"value":{"type":"ref","content":{"ref_node_id":"node_1733450399126","ref_var_name":"key0","source":"user"}}},{"name":"result2","description":"","required":true,"source":"user","reflection":false,"value":{"type":"ref","content":{"ref_node_id":"node_1733450683042","ref_var_name":"raw_output","source":"user"},"hint":""}},{"name":"result3","description":"","required":true,"source":"user","reflection":false,"value":{"type":"ref","content":{"ref_node_id":"node_1733450861668","ref_var_name":"name","source":"user"},"hint":""}},{"name":"result4","description":"","required":true,"source":"user","reflection":false,"value":{"type":"ref","content":{"ref_node_id":"node_1733450399126","ref_var_name":"key0","source":"user"},"hint":""}}],"outputs":[{"name":"response_content","type":"string","description":"最终输出","required":true,"source":"system","reflection":false,"value":{"type":"generated"}}],"configs":{"is_stream_out":true,"response_template":"{{result1}}\n{{result2}}\n{{result3}}\n{{result4}}","response_mode":"directResponse"}},{"id":"node_1733450125254","name":"意图识别","type":"IntentDetection","inputs":[{"name":"query","description":"","required":true,"source":"user","reflection":false,"value":{"type":"ref","content":{"ref_node_id":"node_start","ref_var_name":"query","source":"system"},"hint":""}}],"outputs":[{"name":"result","type":"string","description":"输出的分类分支名","required":false,"source":"system","reflection":false,"value":{"type":"generated","content":"","hint":""}}],"configs":{"llm":{"temperature":0.5,"top_p":0.5,"model":{"model_name":"lx-n2-32k-润达-v3-24112501","model_type":"ei_pangu","model_deployment_id":"6d60c845-3f7d-4925-b9a4-b3d5470d114e"}},"prompt":"你是一个功能分类器，你可以根据用户的请求提问，和相应的功能类别描述，选择正确的功能帮助用户解决问题。"},"branches":[{"id":"branch_1","configs":{"category":"处理数据"}},{"id":"branch_2","configs":{"category":"天气查询"}},{"id":"branch_3","configs":{"category":"登记信息"}},{"id":"branch_4","configs":{"category":"python代码"}}]},{"id":"node_1733450259596","name":"提问器","type":"Questioner","inputs":[{"name":"query","description":"","required":true,"source":"user","reflection":false,"value":{"type":"ref","content":{"ref_node_id":"node_start","ref_var_name":"query","source":"system"},"hint":""}}],"outputs":[{"name":"user_response","cn_name":"","type":"string","description":"用户最近一轮对话输入","required":false,"source":"system","reflection":false,"value":{"default":"","type":"literal","content":"","hint":""},"validator":[]},{"name":"data","cn_name":"数据","type":"string","description":"待处理数据","required":true,"source":"user","reflection":false,"value":{"default":"","type":"generated","hint":""},"validator":[]}],"configs":{"top_p":0.5,"extract_fields_from_response":true,"with_chat_history":false,"extra_prompt_for_fields_extraction":"","temperature":0.2,"max_response":3,"question_content":"请输入要处理的原始数据","model":{"model_name":"lx-n2-32k-润达-v3-24112501","model_type":"ei_pangu","model_deployment_id":"6d60c845-3f7d-4925-b9a4-b3d5470d114e"}}},{"id":"node_1733450399126","name":"代码","type":"Code","inputs":[{"name":"input","description":"","required":true,"source":"user","reflection":false,"value":{"type":"ref","content":{"ref_node_id":"node_1733450259596","ref_var_name":"data","source":"user"},"hint":""}}],"outputs":[{"name":"key0","type":"string","description":"输出结果","required":false,"source":"user","reflection":false,"value":{"default":"","type":"generated","hint":""}}],"configs":{"code":"defsort_list(str_list):\nlist_result=eval(str_list)\nreturnstr(sorted(list_result))\n\ndefmain(args:dict)->dict:\ninput_str=args.get('input','default'),\noutput=sort_list(input_str)\n\nret={\n\"key0\":output,\n\"key1\":\"hi\"\n}\nreturnret"}},{"id":"node_1733450683042","name":"大模型","type":"LLM","inputs":[{"name":"query","type":"string","description":"","required":false,"source":"user","reflection":false,"value":{"type":"ref","content":{"ref_node_id":"node_start","ref_var_name":"query","source":"system"},"hint":""}}],"outputs":[{"name":"raw_output","type":"string","description":"模型原始输出","required":false,"source":"user","reflection":false,"value":{"type":"literal","content":"","hint":""}}],"configs":{"top_p":0.5,"format_instruction":"","response_format":"text","template_content":"{{query}}","temperature":0.5,"model":{"model_name":"lx-n2-32k-润达-v3-24112501","model_type":"ei_pangu","model_deployment_id":"6d60c845-3f7d-4925-b9a4-b3d5470d114e"},"enable_history":false}},{"id":"node_1733450716936","name":"消息","type":"Message","inputs":[{"name":"input","description":"","required":true,"source":"user","reflection":false,"value":{"type":"ref","content":{"ref_node_id":"node_1733450861668","ref_var_name":"name","source":"user"},"hint":""}}],"outputs":[{"name":"result","type":"string","description":"消息输出","required":false,"source":"system","reflection":false,"value":{"type":"generated","content":"","hint":""}}],"configs":{"template":"这个姓名有问题：{{input}}"}},{"id":"node_1733450845836","name":"判断","type":"Branch","branches":[{"id":"if","configs":{"logic":"or","conditions":[{"operator":"in","left":{"name":"","description":"","required":false,"value":{"type":"ref","content":{"ref_node_id":"node_1733450861668","ref_var_name":"name","source":"user"},"hint":""},"source":"user"},"right":{"name":"","description":"","required":false,"value":{"type":"literal","content":"a","hint":""},"source":"user"}},{"operator":"in","left":{"name":"","description":"","required":false,"value":{"type":"ref","content":{"ref_node_id":"node_1733450861668","ref_var_name":"name","source":"user"},"hint":""},"source":"user"},"right":{"name":"","description":"","required":false,"value":{"type":"literal","content":"A","hint":""},"source":"user"}}]}},{"id":"default"}]},{"id":"node_1733450861668","name":"提问器_1","type":"Questioner","inputs":[{"name":"query","description":"","required":true,"source":"user","reflection":false,"value":{"type":"ref","content":{"ref_node_id":"node_start","ref_var_name":"query","source":"system"},"hint":""}}],"outputs":[{"name":"user_response","cn_name":"","type":"string","description":"用户最近一轮对话输入","required":false,"source":"system","reflection":false,"value":{"default":"","type":"literal","content":"","hint":""},"validator":[]},{"name":"name","cn_name":"姓名","type":"string","description":"要登记的姓名","required":true,"source":"user","reflection":false,"value":{"default":"王小明","type":"generated","hint":""},"validator":[]}],"configs":{"top_p":0.5,"extract_fields_from_response":true,"with_chat_history":false,"extra_prompt_for_fields_extraction":"请提取用户输入文本中的姓名，长度不超过10个字符。","temperature":0.2,"max_response":3,"question_content":"请输入您要登记的信息：姓名","model":{"model_name":"lx-n2-32k-润达-v3-24112501","model_type":"ei_pangu","model_deployment_id":"6d60c845-3f7d-4925-b9a4-b3d5470d114e"}}},{"id":"node_1733451357022","name":"消息_1","type":"Message","inputs":[{"name":"input","description":"","required":true,"source":"user","reflection":false,"value":{"type":"ref","content":{"ref_node_id":"node_1733450861668","ref_var_name":"name","source":"user"},"hint":""}}],"outputs":[{"name":"result","type":"string","description":"消息输出","required":false,"source":"system","reflection":false,"value":{"type":"generated","content":"","hint":""}}],"configs":{"template":"这是个好名字：{{input}}"}},{"id":"node_1733497595906","name":"python","type":"Plugin","inputs":[{"name":"code","description":"python","required":true,"source":"user","reflection":false,"value":{"type":"ref","content":{"ref_node_id":"node_start","ref_var_name":"query","source":"system"},"hint":""}}],"outputs":[{"name":"cot_string","type":"string","description":"执行结果","required":true,"source":"user","reflection":false,"value":{"type":"generated","hint":""}}],"configs":{"id":"9f95b66a-acc1-46ca-9300-d2d5ffd77417"}}],"edges":[{"source":"node_1733497595906","target":"node_end"},{"source":"node_1733450125254","target":"node_1733497595906","branch":"branch_4"},{"source":"node_1733451357022","target":"node_end"},{"source":"node_1733450845836","target":"node_1733451357022","branch":"default"},{"source":"node_1733450716936","target":"node_end"},{"source":"node_1733450845836","target":"node_1733450716936","branch":"if"},{"source":"node_1733450861668","target":"node_1733450845836"},{"source":"node_1733450125254","target":"node_1733450861668","branch":"branch_3"},{"source":"node_1733450683042","target":"node_end"},{"source":"node_1733450125254","target":"node_1733450683042","branch":"branch_2"},{"source":"node_1733450399126","target":"node_end"},{"source":"node_1733450259596","target":"node_1733450399126"},{"source":"node_1733450125254","target":"node_1733450259596","branch":"branch_1"},{"source":"node_start","target":"node_1733450125254"}],"layouts":{"node_start":{"x":126,"y":72},"node_end":{"x":2340,"y":162},"node_1733450125254":{"x":684,"y":72},"node_1733450259596":{"x":1480,"y":40},"node_1733450399126":{"x":1910,"y":40},"node_1733450683042":{"x":1908,"y":144},"node_1733450716936":{"x":1926,"y":270},"node_1733450845836":{"x":1512,"y":270},"node_1733450861668":{"x":1260,"y":180},"node_1733451357022":{"x":1910,"y":430},"node_1733497595906":{"x":1692,"y":540}}},"metadata":{"id":"d14b2c45-a982-4eb2-b12d-8db4874e0bf9","name":"工作流节点全覆盖-01_1","code":"workflow_node_completed_01_1","description":"工作流节点全覆盖","avatar":"","status":"draft","visibility":"project","deleted":0,"dsl_path":"workflow/flow/d14b2c45-a982-4eb2-b12d-8db4874e0bf9/d14b2c45-a982-4eb2-b12d-8db4874e0bf9.json","ir_path":"workflow/ir/d14b2c45-a982-4eb2-b12d-8db4874e0bf9/d14b2c45-a982-4eb2-b12d-8db4874e0bf9.json","created_at":1733558342641,"updated_at":1733558756504,"created_by":"lx-1","creator_id":"d5b74c0f6d314137b975007d62622329","updated_by":"lx-1","updater_id":"248424ae5bee479f9301449bcde4b560","project_id":"bc2903d5916b4278a8265f8d7666c0c0","domain_id":"4bb252e66abb40d8b7aa0411da592422","workspace_id":0,"deploy_wf_version":1733558756504,"workflow_type":"chat"},"plugins":[{"tool_id":"9f95b66a-acc1-46ca-9300-d2d5ffd77417","project_id":"9f906d4c3d3d4443ace2eadea0f202cf","tool_display_name":"python","tool_desc":"测试环境python沙箱接口调用","icon":"//test-static-resource.obs.cn-north-7.ulanqab.huawei.com/pangustudio-agent-console/1.0.0.20241130103728/hws/assets/agent-center/images/plugin-default.svg","request_info":{"url":"","method":"POST","headers":{"Content-Type":"application/json"}},"auth_info":{"scope":"USER","domain":"HEADERS","auth_keys":[{"target_name":"X-Auth-Token","source_name":"X-Auth-Token"}]},"input_schema":"{\"type\":\"object\",\"properties\":{\"code\":{\"location\":\"Body\",\"validate_rule\":\"\",\"validate_type\":\"CHAR\",\"validated\":false,\"name_cn\":\"代码\",\"type\":\"string\",\"description\":\"python\"}},\"required\":[\"code\"]}","output_schema":"{\"type\":\"object\",\"properties\":{\"cot_string\":{\"type\":\"string\",\"description\":\"执行结果\"}},\"required\":[\"cot_string\"]}","is_input_list":false,"is_output_list":false,"type":"inner","creator":"op_svc_plm_container6","creator_id":"6e617e51b4a34bb9bc496b3610a55e3b","created_on":"2024-12-0309:12:02","updated_on":"2025-01-0317:23:11"}],"import_type":"workflow","sub_workflows":[]}
```
