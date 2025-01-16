# ModelArts工作流
## 关键概念

### 节点
节点是ModelArts工作流的关键构成，通过连接不同功能的节点，执行ModelArts工作流的一系列操作。
ModelArts工作流的核心节点请查看 《节点说明》。

### 工作流（Workflow）
适用场景：
面向自动化和批处理情景，适合高质量翻译、数据分析、内容生成、电子邮件自动化等应用程序。该类型应用无法对生成的结果进行多轮对话交互。
常见的交互路径：给出指令->生成内容->结束。

## 节点说明
每个节点都是json格式，其属性描述中，括号中的String、Array、Boolean、Object、Number分别对应jsonl文件支持的4种数据类型，用于表明括号前的字段的数据类型，且字段间通过层次关系表示它们间的主从。
每个节点的`"id"`字段的取值必须确保唯一，格式参考"node_1733731697635"，以"node_"开头，后面是13位数字。
在有些场景下，节点中的`"inputs"`和`"outputs"`都可能会包含多个Object对象。
节点中的各字段的实际取值需要根据ModelArts工作流设置，下面给出的是一些样例。

### 节点：`Start`

"开始" 节点是每个ModelArts工作流应用（Chatflow / Workflow）必备的预设节点，为后续ModelArts工作流节点以及应用的正常流转提供必要的初始信息，例如应用使用者所输入的内容。
实际应用中有几点需要注意：

- 在有些场景下`"inputs"`和`"outputs"`中都可能包含多个Object对象

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

**参考样例**

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

**参考样例**

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


**参考样例**

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

"Code"节点支持运行 Python 代码以在ModelArts工作流程中执行数据转换。它可以简化你的工作流程，适用于Arithmetic、JSON transform、文本处理等情景。
该节点极大地增强了开发人员的灵活性，使他们能够在ModelArts工作流程中嵌入自定义的 Python 脚本，并以预设节点无法达到的方式操作变量。

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

**参考样例**

```python
	{
        "id": "node_1733477983207",
        "name": "代码",
        "type": "Code",
        "inputs": [],
        "outputs": [
          {
            "name": "key1",
            "type": "string",
            "description": "输出",
            "required": false,
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

**参考样例**

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
                {
                  "operator": "eq",
                  "left": {
                    "name": "",
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


**参考样例**

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

**参考样例**

```python

```

### 节点：`Plugin`

"Plugin"节点提供已构造好的插件能力。

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

**参考样例**

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

**参考样例**

```python

```

### 节点：`End`

"End"节点定义一个ModelArts工作流程结束的最终输出内容。每一个ModelArts工作流在完整执行后都需要至少一个结束节点，用于输出完整执行的最终结果。
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

**参考样例**

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

## 编排节点
ModelArts工作流内的节点支持串行和并行的连接模式。

### 串行设计
该结构要求节点按照预设顺序依次执行，每个节点需等待前一个节点完成并输出结果后才能开始工作，有助于**确保任务按照逻辑顺序执行。**

例如，在一个采用串行结构设计的“小说生成”AI应用里，用户输入小说风格、节奏和角色后，LLM按照顺序补全小说大纲、小说剧情和结尾；每个节点都基于前一个节点的输出结果展开工作，确保小说的风格一致性。

### 并行设计
该设计模式允许多个节点在同一时间内共同执行，前置节点可以同时触发位于并行结构内的多个节点。并行结构内的节点不存在依赖关系，能够同时执行任务，更好地提升**节点的任务执行效率。**

例如，在某个并行设计的翻译工作流应用内，用户输入源文本触发工作流后，位于并行结构内的节点将共同收到前置节点的流转指令，同时开展多语言的翻译任务，缩短任务的处理耗时。

### 节点间连接
在ModelArts工作流的配置jsonl文件中，节点间的连接关系是通过`"edges"`字段来结构化表示，该字段是`Array`类型，其中每一个item都是一个`Object`,表示2个节点的连接关系，item中的`"source"`表示源节点的`"id"`、`"target"`表示目的节点的`"id"`，如果有多条路径的话，还有一个字段`"branch"`,表示实际连接路径。详细定义可参考如下样例：

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

## ModelArts工作流配置文件
ModelArts工作流配置文件是未格式化jsonl文件，该文件包含**5个必需的关键字**，分别是`"dsl"`、`"metadata"`、`"plugins"`、`"import_type"`、`"sub_workflows"`。
-  `"dsl"`是一个`Object`类型，其中包括**6个必需的关键字**，分别是`"id"`、`"name"`、`"description"`、`"nodes"`、`"edges"`、`"layouts"`。
- `"metadata"`是一个`Object`类型，其中包括**21个必需的关键字**，分别是`"id"`、`"name"`、`"description"`、`"code"`、`"avatar"`、`"status"`、`"visibility"`、`"deleted"`、`"dsl_path"`、`"ir_path"`、`"created_at"`、`"updated_at"`、`"created_by"`、`"creator_id"`、`"updated_by"`、`"updater_id"`、`"project_id"`、`"domain_id"`、`"deploy_wf_version"`、`"workflow_type"`。
- `"plugins"`是一个`Array`类型，其中每个item都是一个`Object`对象，包括**16个必需的关键字**，分别是`"tool_id"`、`"project_id"`、`"tool_display_name"`、`"tool_desc"`、`"icon"`、`"request_info"`、`"auth_info"`、`"input_schema"`、`"output_schema"`、`"is_input_list"`、`"is_output_list"`、`"type"`、`"creator"`、`"creator_id"`、`"created_on"`、`"updated_on"`。注意，`"dsl"`中的`"nodes"`有几个plugins节点，，这里就有对应数目的item，且这里的`"tool_display_name"`必须和`"dsl"`中的`"nodes"`里面的`"Plugin"`类型的节点中的`"name"`一一对应上。
- `"import_type"`是一个`String`类型的对象，默认值为`"workflow"`
- `"sub_workflows"`是一个`Array`类型的对象，默认值为`[]`

下面是一个配置文件的参考格式样例：
```python
{"dsl":{"id":"d14b2c45-a982-4eb2-b12d-8db4874e0bf9","name":"工作流节点全覆盖-01_1","description":"工作流节点全覆盖","nodes":[{"id":"node_start","name":"开始","type":"Start","outputs":[{"name":"query","type":"string","description":"用户输入","required":true,"source":"system","field_type":"input","reflection":false,"value":{"type":"literal","content":"","hint":"用户输入"}}]},{"id":"node_end","name":"结束","type":"End","inputs":[{"name":"result1","type":"string","description":"最终输出","required":false,"source":"user","reflection":false,"value":{"type":"ref","content":{"ref_node_id":"node_1733450399126","ref_var_name":"key0","source":"user"}}},{"name":"result2","description":"","required":true,"source":"user","reflection":false,"value":{"type":"ref","content":{"ref_node_id":"node_1733450683042","ref_var_name":"raw_output","source":"user"},"hint":""}},{"name":"result3","description":"","required":true,"source":"user","reflection":false,"value":{"type":"ref","content":{"ref_node_id":"node_1733450861668","ref_var_name":"name","source":"user"},"hint":""}},{"name":"result4","description":"","required":true,"source":"user","reflection":false,"value":{"type":"ref","content":{"ref_node_id":"node_1733450399126","ref_var_name":"key0","source":"user"},"hint":""}}],"outputs":[{"name":"response_content","type":"string","description":"最终输出","required":true,"source":"system","reflection":false,"value":{"type":"generated"}}],"configs":{"is_stream_out":true,"response_template":"{{result1}}\n{{result2}}\n{{result3}}\n{{result4}}","response_mode":"directResponse"}},{"id":"node_1733450125254","name":"意图识别","type":"IntentDetection","inputs":[{"name":"query","description":"","required":true,"source":"user","reflection":false,"value":{"type":"ref","content":{"ref_node_id":"node_start","ref_var_name":"query","source":"system"},"hint":""}}],"outputs":[{"name":"result","type":"string","description":"输出的分类分支名","required":false,"source":"system","reflection":false,"value":{"type":"generated","content":"","hint":""}}],"configs":{"llm":{"temperature":0.5,"top_p":0.5,"model":{"model_name":"lx-n2-32k-润达-v3-24112501","model_type":"ei_pangu","model_deployment_id":"6d60c845-3f7d-4925-b9a4-b3d5470d114e"}},"prompt":"你是一个功能分类器，你可以根据用户的请求提问，和相应的功能类别描述，选择正确的功能帮助用户解决问题。"},"branches":[{"id":"branch_1","configs":{"category":"处理数据"}},{"id":"branch_2","configs":{"category":"天气查询"}},{"id":"branch_3","configs":{"category":"登记信息"}},{"id":"branch_4","configs":{"category":"python代码"}}]},{"id":"node_1733450259596","name":"提问器","type":"Questioner","inputs":[{"name":"query","description":"","required":true,"source":"user","reflection":false,"value":{"type":"ref","content":{"ref_node_id":"node_start","ref_var_name":"query","source":"system"},"hint":""}}],"outputs":[{"name":"user_response","cn_name":"","type":"string","description":"用户最近一轮对话输入","required":false,"source":"system","reflection":false,"value":{"default":"","type":"literal","content":"","hint":""},"validator":[]},{"name":"data","cn_name":"数据","type":"string","description":"待处理数据","required":true,"source":"user","reflection":false,"value":{"default":"","type":"generated","hint":""},"validator":[]}],"configs":{"top_p":0.5,"extract_fields_from_response":true,"with_chat_history":false,"extra_prompt_for_fields_extraction":"","temperature":0.2,"max_response":3,"question_content":"请输入要处理的原始数据","model":{"model_name":"lx-n2-32k-润达-v3-24112501","model_type":"ei_pangu","model_deployment_id":"6d60c845-3f7d-4925-b9a4-b3d5470d114e"}}},{"id":"node_1733450399126","name":"代码","type":"Code","inputs":[{"name":"input","description":"","required":true,"source":"user","reflection":false,"value":{"type":"ref","content":{"ref_node_id":"node_1733450259596","ref_var_name":"data","source":"user"},"hint":""}}],"outputs":[{"name":"key0","type":"string","description":"输出结果","required":false,"source":"user","reflection":false,"value":{"default":"","type":"generated","hint":""}}],"configs":{"code":"defsort_list(str_list):\nlist_result=eval(str_list)\nreturnstr(sorted(list_result))\n\ndefmain(args:dict)->dict:\ninput_str=args.get('input','default'),\noutput=sort_list(input_str)\n\nret={\n\"key0\":output,\n\"key1\":\"hi\"\n}\nreturnret"}},{"id":"node_1733450683042","name":"大模型","type":"LLM","inputs":[{"name":"query","type":"string","description":"","required":false,"source":"user","reflection":false,"value":{"type":"ref","content":{"ref_node_id":"node_start","ref_var_name":"query","source":"system"},"hint":""}}],"outputs":[{"name":"raw_output","type":"string","description":"模型原始输出","required":false,"source":"user","reflection":false,"value":{"type":"literal","content":"","hint":""}}],"configs":{"top_p":0.5,"format_instruction":"","response_format":"text","template_content":"{{query}}","temperature":0.5,"model":{"model_name":"lx-n2-32k-润达-v3-24112501","model_type":"ei_pangu","model_deployment_id":"6d60c845-3f7d-4925-b9a4-b3d5470d114e"},"enable_history":false}},{"id":"node_1733450716936","name":"消息","type":"Message","inputs":[{"name":"input","description":"","required":true,"source":"user","reflection":false,"value":{"type":"ref","content":{"ref_node_id":"node_1733450861668","ref_var_name":"name","source":"user"},"hint":""}}],"outputs":[{"name":"result","type":"string","description":"消息输出","required":false,"source":"system","reflection":false,"value":{"type":"generated","content":"","hint":""}}],"configs":{"template":"这个姓名有问题：{{input}}"}},{"id":"node_1733450845836","name":"判断","type":"Branch","branches":[{"id":"if","configs":{"logic":"or","conditions":[{"operator":"in","left":{"name":"","description":"","required":false,"value":{"type":"ref","content":{"ref_node_id":"node_1733450861668","ref_var_name":"name","source":"user"},"hint":""},"source":"user"},"right":{"name":"","description":"","required":false,"value":{"type":"literal","content":"a","hint":""},"source":"user"}},{"operator":"in","left":{"name":"","description":"","required":false,"value":{"type":"ref","content":{"ref_node_id":"node_1733450861668","ref_var_name":"name","source":"user"},"hint":""},"source":"user"},"right":{"name":"","description":"","required":false,"value":{"type":"literal","content":"A","hint":""},"source":"user"}}]}},{"id":"default"}]},{"id":"node_1733450861668","name":"提问器_1","type":"Questioner","inputs":[{"name":"query","description":"","required":true,"source":"user","reflection":false,"value":{"type":"ref","content":{"ref_node_id":"node_start","ref_var_name":"query","source":"system"},"hint":""}}],"outputs":[{"name":"user_response","cn_name":"","type":"string","description":"用户最近一轮对话输入","required":false,"source":"system","reflection":false,"value":{"default":"","type":"literal","content":"","hint":""},"validator":[]},{"name":"name","cn_name":"姓名","type":"string","description":"要登记的姓名","required":true,"source":"user","reflection":false,"value":{"default":"王小明","type":"generated","hint":""},"validator":[]}],"configs":{"top_p":0.5,"extract_fields_from_response":true,"with_chat_history":false,"extra_prompt_for_fields_extraction":"请提取用户输入文本中的姓名，长度不超过10个字符。","temperature":0.2,"max_response":3,"question_content":"请输入您要登记的信息：姓名","model":{"model_name":"lx-n2-32k-润达-v3-24112501","model_type":"ei_pangu","model_deployment_id":"6d60c845-3f7d-4925-b9a4-b3d5470d114e"}}},{"id":"node_1733451357022","name":"消息_1","type":"Message","inputs":[{"name":"input","description":"","required":true,"source":"user","reflection":false,"value":{"type":"ref","content":{"ref_node_id":"node_1733450861668","ref_var_name":"name","source":"user"},"hint":""}}],"outputs":[{"name":"result","type":"string","description":"消息输出","required":false,"source":"system","reflection":false,"value":{"type":"generated","content":"","hint":""}}],"configs":{"template":"这是个好名字：{{input}}"}},{"id":"node_1733497595906","name":"python","type":"Plugin","inputs":[{"name":"code","description":"python","required":true,"source":"user","reflection":false,"value":{"type":"ref","content":{"ref_node_id":"node_start","ref_var_name":"query","source":"system"},"hint":""}}],"outputs":[{"name":"cot_string","type":"string","description":"执行结果","required":true,"source":"user","reflection":false,"value":{"type":"generated","hint":""}}],"configs":{"id":"9f95b66a-acc1-46ca-9300-d2d5ffd77417"}}],"edges":[{"source":"node_1733497595906","target":"node_end"},{"source":"node_1733450125254","target":"node_1733497595906","branch":"branch_4"},{"source":"node_1733451357022","target":"node_end"},{"source":"node_1733450845836","target":"node_1733451357022","branch":"default"},{"source":"node_1733450716936","target":"node_end"},{"source":"node_1733450845836","target":"node_1733450716936","branch":"if"},{"source":"node_1733450861668","target":"node_1733450845836"},{"source":"node_1733450125254","target":"node_1733450861668","branch":"branch_3"},{"source":"node_1733450683042","target":"node_end"},{"source":"node_1733450125254","target":"node_1733450683042","branch":"branch_2"},{"source":"node_1733450399126","target":"node_end"},{"source":"node_1733450259596","target":"node_1733450399126"},{"source":"node_1733450125254","target":"node_1733450259596","branch":"branch_1"},{"source":"node_start","target":"node_1733450125254"}],"layouts":{"node_start":{"x":126,"y":72},"node_end":{"x":2340,"y":162},"node_1733450125254":{"x":684,"y":72},"node_1733450259596":{"x":1480,"y":40},"node_1733450399126":{"x":1910,"y":40},"node_1733450683042":{"x":1908,"y":144},"node_1733450716936":{"x":1926,"y":270},"node_1733450845836":{"x":1512,"y":270},"node_1733450861668":{"x":1260,"y":180},"node_1733451357022":{"x":1910,"y":430},"node_1733497595906":{"x":1692,"y":540}}},"metadata":{"id":"d14b2c45-a982-4eb2-b12d-8db4874e0bf9","name":"工作流节点全覆盖-01_1","code":"workflow_node_completed_01_1","description":"工作流节点全覆盖","avatar":"","status":"draft","visibility":"project","deleted":0,"dsl_path":"workflow/flow/d14b2c45-a982-4eb2-b12d-8db4874e0bf9/d14b2c45-a982-4eb2-b12d-8db4874e0bf9.json","ir_path":"workflow/ir/d14b2c45-a982-4eb2-b12d-8db4874e0bf9/d14b2c45-a982-4eb2-b12d-8db4874e0bf9.json","created_at":1733558342641,"updated_at":1733558756504,"created_by":"lx-1","creator_id":"d5b74c0f6d314137b975007d62622329","updated_by":"lx-1","updater_id":"248424ae5bee479f9301449bcde4b560","project_id":"bc2903d5916b4278a8265f8d7666c0c0","domain_id":"4bb252e66abb40d8b7aa0411da592422","workspace_id":0,"deploy_wf_version":1733558756504,"workflow_type":"chat"},"plugins":[{"tool_id":"9f95b66a-acc1-46ca-9300-d2d5ffd77417","project_id":"9f906d4c3d3d4443ace2eadea0f202cf","tool_display_name":"python","tool_desc":"测试环境python沙箱接口调用","icon":"//test-static-resource.obs.cn-north-7.ulanqab.huawei.com/pangustudio-agent-console/1.0.0.20241130103728/hws/assets/agent-center/images/plugin-default.svg","request_info":{"url":"","method":"POST","headers":{"Content-Type":"application/json"}},"auth_info":{"scope":"USER","domain":"HEADERS","auth_keys":[{"target_name":"X-Auth-Token","source_name":"X-Auth-Token"}]},"input_schema":"{\"type\":\"object\",\"properties\":{\"code\":{\"location\":\"Body\",\"validate_rule\":\"\",\"validate_type\":\"CHAR\",\"validated\":false,\"name_cn\":\"代码\",\"type\":\"string\",\"description\":\"python\"}},\"required\":[\"code\"]}","output_schema":"{\"type\":\"object\",\"properties\":{\"cot_string\":{\"type\":\"string\",\"description\":\"执行结果\"}},\"required\":[\"cot_string\"]}","is_input_list":false,"is_output_list":false,"type":"inner","creator":"op_svc_plm_container6","creator_id":"6e617e51b4a34bb9bc496b3610a55e3b","created_on":"2024-12-0309:12:02","updated_on":"2025-01-0317:23:11"}],"import_type":"workflow","sub_workflows":[]}
```
