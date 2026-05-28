# Agent AIB

# 框架：

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/ZWGl05mxmDJXZn34/img/736f6592-62c6-4d2a-9d7e-34bc217199c0.png)

结合业界经典的Plan-and-Execute和ReAct范式，我们设计了一套“规划-执行”的双循环架构，Planner拆解用户目标为子任务，Reasoner结合工具与策略生成行动，Executor调用工具或输出结果。

1.Planner（规划器） Planner负责将用户的目标拆解为具体的任务（Task）。例如，如果用户询问“我的订单怎么还没发货？”，Planner可能会将其分解为“确认订单状态”、“查找物流信息”和“安抚用户情绪”等子任务。

2.Reasoner（推理器） Reasoner是整个Agent的核心，负责执行具体的任务并产生行动指令（Action）。例如，针对“确认订单状态”的任务，Reasoner可能决定调用订单查询工具，或者直接生成一段安抚用户的文字。

3.Executor（动作执行器） Executor负责执行由Reasoner生成的Action。这些Action可以是调用工具、输出结果、重新规划或转交任务等。

4.Tools（工具） Tools是Agent感知和改变环境的接口，例如查询用户状态、检索知识库或播放解决方案等。Reasoner会根据当前任务的需求选择合适的工具及其参数。

执行流程如下： 1.用户输入Query后，Planner首先对其进行初步规划，生成具体的Task列表。 2.Task被传递至Reasoner，后者结合服务策略生成Action列表。 3.Action列表交由Executor逐条执行。如果需要依赖外部信息，Executor会调用相应工具并将结果反馈给Reasoner进行下一轮推理。 4.当所有Action执行完毕后，最终结果通过Output类型Action返回给用户。

### Multi- agent框架：

售前、售后-方案前、售后-方案后。三个agent互相转交的整体方案。

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/ZWGl05mxmDJXZn34/img/d4604adb-d2eb-4185-85c5-427afb2b037e.png)

防止死循环 为了避免Agent之间的频繁转交导致死循环，我们设置了两项约束： 1.重复转交限制：每个Agent只能将任务转交给未参与过的Agent。 2.转交次数上限：每轮对话最多允许一次转交，超过限制的Agent必须自行处理问题。 通过以上设计，我们实现了专业化分工与高效协作的平衡，同时保证了对话的连贯性和拟人化效果。

是品

### 服务策略skills抽象：

对话链路升级到Agent 版后，基于大模型的对话能力得到有效提升，如何让运营有效优化Agent效果，我们在Agent工程链路之上抽象了服务策略层。服务策略本质是LLM提示词的一部分，我们把需要业务运营的部分单独抽象出来，在Agent 链路里通过召回+上下文组装方式给到大模型

参考小二服务服务过程依赖要素，我们抽象服务策略skills 能力如下：

●服务策略：类似小二服务思路，指导Agent应该如何服务会员问题

●约束限制：明确约束限制大模型不能做的事情

●动态视图：参考小二服务过程需要查询的视图信息，本质是数据特征

●ISO能力: 这个问题大模型能使用的解决能力列表

●知识库：这个会员问题对应的常识

![截屏2026-05-25 17.11.57.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/ZWGl05mxmDJXZn34/img/c0c60ccf-5c35-4c53-b30b-ce3923ee311f.png)

是品

### ISO = 工单操作能力（Issue Solving Operations）

在电商客服"小二服务"场景中，小二处理会员问题时，除了"说"（话术/策略），还需要实际执行操作，例如：

| 小二实际操作 | 对应 ISO 能力 |
| --- | --- |
| 帮用户申请退款 | 退款工具调用 |
| 发放优惠补偿券 | 发券接口 |
| 修改订单信息 | 订单修改工具 |
| 催促物流发货 | 物流催单接口 |
| 转接人工客服 | 转人工能力 |

ISO 能力本质 = Agent 在该问题场景下，可以调用的工具（Tool）列表

服务策略 → "怎么说"（指导方向） 约束限制 → "不能做什么"（边界） 动态视图 → "看什么数据"（信息输入） ISO 能力 → "能做什么操作"（执行动作 知识库 → "知道什么"（背景知识）

### 主要挑战：

##### RT问题

# AIDC-小蜜框架

### 中枢Agent详细设计

原有架构下以小模型单轮意图识别为准，无多轮上下文识别能力，多轮对话以SOP能力承载，缺乏灵活性。新架构构建中枢Agent替代原有意图识别模型，提升意图定位与多轮交互能力，通过Agent能力与用户进行开场问题澄清与意图识别，并作为中枢将识别到的明确用户意图转交给对应场景的解决方案Agent来承接对话。

﻿![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/ZWGl05mxmDJXZn34/img/22c5ed55-91fa-4c3f-93ab-b1a5f052eb69)﻿

中枢 Agent 模仿人工小二开场能力，我们在中枢 Agent 加入订单和问题预测能力，辅助 Agent 确认用户订单和问题，并从长期记忆中加载用户聊天记录，防止用户进线之后重复描述问题，帮助 Agent 更快定位用户意图。

### 3.2.4 解决方案Agent详细设计

原有机器人播放解决方案依赖业务运营配置的服务SOP，采用严格线性执行机制，节点推进完全依赖预设顺序，无法适应用户意图动态变更。当用户表达跳跃性需求时，系统无法回溯至历史节点或跳过冗余人工因子，导致重复询问已提供的信息，同时缺乏上下文语义理解能力，无法准确解析代词指代、省略表达等真实对话特征，需用户多次澄清意图，服务流程死板且用户体感僵硬。

在新架构下，我们参考小二技能组划分模式，将服务域场景进行划分，针对每个业务场景构建解决方案子Agent，解决方案Agent主要分为基于SOP结构转义实现和基于SOP播放两种范式实现的能力，下面详细介绍两种方案：

#### 3.2.4.1 基于SOP转义的解决方案Agent

在复杂业务场景下，我们采用了基于SOP流程转义的方案来构建解决方案Agent，以SOP流程为核心业务规则，将现有结构化的SOP配置进行转义成伪代码交给模型理解，让Agent可以通过阅读SOP转义后的伪代码掌握业务规则，在SOP播放时基于当前播放的节点设置滑动窗口，动态获取SOP片段的转义代码，模型可以感知到未来N步用户可能咨询的场景，保证模型的灵活性。

﻿![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/ZWGl05mxmDJXZn34/img/523c5426-3239-4c0e-a0a3-c0537589263f)﻿

#### 3.2.4.2 基于SOP播放的解决方案Agent

在中小业务场景下，我们提供了以驱动SOP播放为核心的解决方案Agent，在客服小二实际服务客户过程中播放SOP时，小二会与客户沟通获取推进SOP流程所需要的必要信息，在用户切换意图时，也可以回溯到已经播放过的SOP节点，通过更换SOP分支播放另外的解决方案流程，参考小二操作流程，以SOP驱动为核心，辅以大模型的理解沟通能力，我们设计了一套基于SOP播放的解决方案Agent。

﻿![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/ZWGl05mxmDJXZn34/img/5faf9eb0-d825-4209-a6c6-ea4d792bd72d)﻿

参考开源Agent框架[Parlant](https://www.zdoc.app/zh/emcie-co/parlant?spm=defwork..0.0.62c522c1rz3Pp2)实现原理，梳理SOP节点并语义化，在实际运行中将SOP流程定义为Journey存入Agent上下文中，辅助Agent理解SOP流程，基于对当前Journey的理解，通过与用户对话获取推动SOP继续推进的必要信息，实现从“机械播放SOP”向“理解并动态驱动SOP”的升级。

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/ZWGl05mxmDJXZn34/img/17a91bfd-8005-4064-a56d-764e1312996b.png)

### 旁路Agent详细设计

旁路 Agent 作为对话流程的监督者，主要负责以下职责：

1.高危场景监控：实时监控用户对话内容，当检测到高危场景（如自杀、涉政涉暴等极端问题）时，立即接管会话并直接转接人工服务，确保风险可控。

2.会话流畅性保障：在主链路 Agent 无法解答用户问题时及时介入，提供兜底回答或转人工处理，避免对话中断或用户等待过久。

3.转人工逻辑控制：通过规则+算法模型的能力相结合，在Agent服务过程中，动态调控用户进人工的入口，保证人工资源最大化利用。

4.主动追问能力：当用户长时间不回复时，主动询问用户是否还有其他问题，模拟小二真实服务场景，提升机器人拟人程度。

5.结束语处理：当用户发送"谢谢"、"再见"等结束语时，基于上下文流畅衔接，提供自然的结束对话体验。

﻿![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/ZWGl05mxmDJXZn34/img/0f2f3e5c-de6e-42d7-aacc-fc9ce963de3d)﻿

### 3.2.6 风控Agent详细设计

风控 Agent 在答案输出环节进行内容安全校验，确保所有对外输出的答案符合合规要求。其工作流程如下：

1.答案接收：接收来自解决方案 Agent 生成的候选答案。

2.维度校验：从内容安全、业务合规、用户体验等多个维度对答案进行全面评估。

3.风险识别：识别答案中可能存在的敏感信息、不当表述、违规承诺等风险点。

4.反馈修正：如果答案不合法，输出不合法的原因并打回给生成答案的 Agent 重新生成，直到输出的答案满足风控要求。

﻿![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/ZWGl05mxmDJXZn34/img/b5b5de47-cadd-484a-9c03-76dc1e314a59)﻿

# 业务指标&模型优化指标

### 业务指标：

1.智能解决率 

2. 满意度 

### 模型优化指标：

准确率 / 召回率

SOP agent SOP Agent + 工具调用评估：  输入 → \[节点1\] → \[工具A\] → \[节点2\] → \[工具B\] → \[节点3\] → 输出    需要同时评估：  ① SOP路径是否正确 （路径维度）  ② 工具调用是否正确 （工具维度）  ③ 工具调用时机是否正确 （时序维度）  ④ 最终结果是否正确 （结果维度）  ⑤ 综合

├── 1. SOP路径维度 │ ├── 硬SOP遵循率 │ └── 软SOP跳过决策准确率 ├── 2. 工具调用维度 │ ├── 工具选择准确率 │ ├── 工具参数准确率 │ └── 工具调用时机准确率 ├── 3. 时序维度 │ ├── 节点顺序正确率 │ └── 工具与节点的配合正确率 ├── 4. 结果维度 │ ├── 任务完成率 │ └── 结果质量分

1.  SOP不遵守路径该怎么办
    
    业务视角：
    
    设计层面：SOP 流程约束没有被有效编码进系统
    
    技术视角：
    
    模型层面：LLM 的生成具有随机性，不天然遵循约定 数据层面：训练/微调数据不够或质量差 工程层面：缺乏硬性约束和兜底机制
    
    解决方案 ├── 1. Prompt 层面 ├── 2. 流程控制层面（硬约束） ├── 3. 模型训练层面 ├── 4. 检测与纠偏层面 └── 5. 评估与监控层面
    
    ## 三、各层面详细方案
    
    ### 3.1 Prompt 层面
    
    显式约束 SOP 路径
    
    ```plaintext
    方案1：结构化 Prompt 注入 SOP
      将 SOP 流程直接写入 System Prompt：
      
      "你必须严格按照以下步骤执行：
       Step1：先询问用户订单号
       Step2：验证订单状态
       Step3：根据状态选择对应话术
       ⚠️ 禁止跳过任何步骤"
    
    方案2：当前状态注入
      每次调用时告诉模型"当前在哪个节点"：
      
      "当前节点：订单查询
       下一步只能是：[确认身份] 或 [查询失败处理]
       禁止执行其他操作"
    
    方案3：Few-shot 示例
      提供标准 SOP 执行示例，
      让模型学习正确的路径选择
    
    ```
    ---
    
    ### 3.2 流程控制层面（硬约束）★ 最重要
    
    用代码强制控制，而非依赖模型自觉
    
    ```python
    python# 有限状态机（FSM）控制 SOP 流程
    
    class SOPStateMachine:
        def __init__(self):
            # 定义合法的状态转移
            self.valid_transitions = {
                "start":           ["verify_identity"],
                "verify_identity": ["query_order", "identity_failed"],
                "query_order":     ["order_found", "order_not_found"],
                "order_found":     ["handle_complaint", "handle_refund"],
                "order_not_found": ["transfer_human"],
                # ...
            }
            self.current_state = "start"
        
        def can_transition(self, next_state: str) -> bool:
            """硬性检查：是否允许跳转"""
            allowed = self.valid_transitions.get(self.current_state, [])
            return next_state in allowed
        
        def transition(self, next_state: str):
            if not self.can_transition(next_state):
                # 拒绝非法跳转，强制走默认路径
                raise IllegalTransitionError(
                    f"不允许从 {self.current_state} 跳转到 {next_state}"
                )
            self.current_state = next_state
    ```
    ```plaintext
    核心思想：
      LLM 负责理解用户意图 → 输出"想去哪个节点"
      FSM 负责校验是否合法 → 决定"能不能去"
      
      LLM 只有"建议权"，FSM 有"否决权"
    
    ```
    
    节点级别的输出约束
    
    ```python
    python# 每个节点限制 LLM 的输出选项
    def get_node_prompt(current_node: str) -> str:
        node_config = {
            "verify_identity": {
                "task": "验证用户身份",
                "allowed_outputs": ["identity_verified", "identity_failed"],
                "prompt": "你只能回复以下之一：[通过验证] 或 [验证失败]"
            }
        }
        return node_config[current_node]
    ```
    ---
    
    ### 3.3 模型训练层面
    
    ```plaintext
    方案1：SFT 微调
      收集大量正确 SOP 执行轨迹
      → 微调模型，让其学会正确的路径选择
      
      数据构造：
      input:  用户说"我要退款" + 当前在[订单查询]节点
      output: 跳转到[退款处理]节点（而非其他节点）
    
    方案2：RLHF / GRPO 强化学习
      定义奖励函数：
      
      r = +1  如果按 SOP 路径走
      r = -1  如果偏离 SOP 路径
      r = +2  如果最终解决了用户问题
      
      用强化学习让模型学会"遵守 SOP 同时解决问题"
    
    方案3：DPO（直接偏好优化）
      构造偏好对：
      chosen:   正确 SOP 路径的回复
      rejected: 偏离 SOP 路径的回复
      → 训练模型偏好正确路径
    
    ```
    ---
    
    ### 3.4 检测与纠偏层面
    
    ```python
    pythonclass SOPGuardrail:
        """
        实时检测 Agent 输出是否偏离 SOP
        """
        
        def check_and_correct(self, 
                               agent_output: str,
                               current_node: str,
                               context: dict) -> str:
            
            # Step1：检测意图
            intended_next = self.extract_intent(agent_output)
            
            # Step2：校验合法性
            if not self.fsm.can_transition(intended_next):
                
                # Step3：纠偏策略
                corrected = self.correct(
                    current_node=current_node,
                    illegal_target=intended_next,
                    context=context
                )
                return corrected
            
            return agent_output
        
        def correct(self, current_node, illegal_target, context):
            """纠偏：强制回到正确路径"""
            
            # 策略1：回到当前节点重新执行
            return self.re_execute_node(current_node, context)
            
            # 策略2：走默认路径
            # return self.default_transition(current_node)
            
            # 策略3：转人工
            # return self.transfer_to_human(context)
    ```
    
    自我反思机制（Self-Reflection）
    
    ```plaintext
    在 Agent 输出后，加一个"检查员 LLM"：
    
    检查员 Prompt：
    "当前 SOP 节点是[订单查询]
     Agent 准备执行[直接退款]
     这是否符合 SOP 规定？
     如果不符合，应该走哪个节点？"
    
    → 两层 LLM 互相校验，减少偏离概率
    
    ```
    ---
    
    ### 3.5 评估与监控层面
    
    ```plaintext
    离线评估：
      构建 SOP 遵循率评估集
      定期评测 Agent 的路径遵循情况
      
      指标：
      - SOP 遵循率 = 正确转移次数 / 总转移次数
      - 任务完成率
      - 平均偏离节点数
    
    在线监控：
      实时记录每次状态转移
      设置告警阈值（遵循率 < 90% 触发告警）
      异常轨迹自动标记，人工复查
    
    数据飞轮：
      偏离案例 → 人工标注正确路径
               → 加入训练数据
               → 重新微调模型
               → 遵循率提升
    
    ```
    ---
    
    ## 四、方案优先级建议
    
    ```plaintext
    优先级排序（从最有效到辅助）：
    
    ★★★  有限状态机硬约束    → 从根本上保证不偏离
    ★★★  节点级输出限制      → 每步只能选合法动作
    
    ★★   Prompt 注入 SOP    → 软约束，成本低
    ★★   自我反思检查        → 加一层校验
    
    ★    SFT 微调            → 长期效果好，成本高
    ★    强化学习            → 效果最好，实现复杂
    
    辅助  监控评估体系        → 发现问题，持续改进
    ```
    
2.  RT问题，如何解决高RT的问题
    

架构：
