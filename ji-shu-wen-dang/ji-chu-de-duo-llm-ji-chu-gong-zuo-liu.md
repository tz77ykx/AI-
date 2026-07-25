# 基础的多 LLM 基础工作流

三种简单的多 LLM 工作流模式，通过成本或延迟来换取可能更好的任务性能。

[在 GitHub 上查看](https://github.com/anthropics/claude-cookbooks/blob/main/patterns/agents/basic_workflows.ipynb)

## 多 LLM 基础工作流

> **译者注**：本文是一份可运行的代码食谱（cookbook）。为保持示例的可复现性，代码中的提示字符串（prompt strings）和 LLM 生成的示例输出仍保留英文；说明性文字、代码注释和 docstring 已译为中文。

本文档演示三种简单的多 LLM 工作流。它们以成本或延迟为代价，用于换取可能更好的任务性能：

1. **提示链（Prompt-Chaining）**：将任务分解为顺序子任务，每一步都基于前面的结果
2. **并行化（Parallelization）**：将独立的子任务分发给多个 LLM 并发处理
3. **路由（Routing）**：根据输入特征动态选择专门化的 LLM 处理路径

注意：这些是实现核心概念的示例代码，并非生产代码。

```python
from concurrent.futures import ThreadPoolExecutor

from util import extract_xml, llm_call


def chain(input: str, prompts: list[str]) -> str:
    """按顺序链式调用多个 LLM，在各步骤之间传递结果。"""
    result = input
    for i, prompt in enumerate(prompts, 1):
        print(f"\nStep {i}:")
        result = llm_call(f"{prompt}\nInput: {result}")
        print(result)
    return result


def parallel(prompt: str, inputs: list[str], n_workers: int = 3) -> list[str]:
    """使用相同的提示并发处理多个输入。"""
    with ThreadPoolExecutor(max_workers=n_workers) as executor:
        futures = [executor.submit(llm_call, f"{prompt}\nInput: {x}") for x in inputs]
        return [f.result() for f in futures]


def route(input: str, routes: dict[str, str]) -> str:
    """基于内容分类将输入路由到专门化的提示。"""
    # 首先使用带思维链（chain-of-thought）的 LLM 确定合适的路由
    print(f"\nAvailable routes: {list(routes.keys())}")
    selector_prompt = f"""
    Analyze the input and select the most appropriate support team from these options: {list(routes.keys())}
    First explain your reasoning, then provide your selection in this XML format:

    <reasoning>
    Brief explanation of why this ticket should be routed to a specific team.
    Consider key terms, user intent, and urgency level.
    </reasoning>

    <selection>
    The chosen team name
    </selection>

    Input: {input}""".strip()

    route_response = llm_call(selector_prompt)
    reasoning = extract_xml(route_response, "reasoning")
    route_key = extract_xml(route_response, "selection").strip().lower()

    print("Routing Analysis:")
    print(reasoning)
    print(f"\nSelected route: {route_key}")

    # 使用选定的专门化提示处理输入
    selected_prompt = routes[route_key]
    return llm_call(f"{selected_prompt}\nInput: {input}")
```

## 示例用法

下面是演示每种工作流的实用示例：

1. 用于结构化数据提取与格式化的链式工作流
2. 用于利益相关方影响分析的并行化工作流
3. 用于客户支持工单处理的路由工作流

### 示例 1：用于结构化数据提取与格式化的链式工作流

每一步都逐步将原始文本转换为格式化的表格。

```python
# 示例 1：用于结构化数据提取与格式化的链式工作流
# 每一步都逐步将原始文本转换为格式化的表格

data_processing_steps = [
    """Extract only the numerical values and their associated metrics from the text.
    Format each as 'value: metric' on a new line.
    Example format:
    92: customer satisfaction
    45%: revenue growth""",
    """Convert all numerical values to percentages where possible.
    If not a percentage or points, convert to decimal (e.g., 92 points -> 92%).
    Keep one number per line.
    Example format:
    92%: customer satisfaction
    45%: revenue growth""",
    """Sort all lines in descending order by numerical value.
    Keep the format 'value: metric' on each line.
    Example:
    92%: customer satisfaction
    87%: employee satisfaction""",
    """Format the sorted data as a markdown table with columns:
    | Metric | Value |
    |:--|--:|
    | Customer Satisfaction | 92% |""",
]

report = """
Q3 Performance Summary:
Our customer satisfaction score rose to 92 points this quarter.
Revenue grew by 45% compared to last year.
Market share is now at 23% in our primary market.
Customer churn decreased to 5% from 8%.
New user acquisition cost is $43 per user.
Product adoption rate increased to 78%.
Employee satisfaction is at 87 points.
Operating margin improved to 34%.
"""

print("\nInput text:")
print(report)
formatted_result = chain(report, data_processing_steps)
```

输出：

```
Input text:

Q3 Performance Summary:
Our customer satisfaction score rose to 92 points this quarter.
Revenue grew by 45% compared to last year.
Market share is now at 23% in our primary market.
Customer churn decreased to 5% from 8%.
New user acquisition cost is $43 per user.
Product adoption rate increased to 78%.
Employee satisfaction is at 87 points.
Operating margin improved to 34%.


Step 1:
92: customer satisfaction points
45%: revenue growth
23%: market share
5%: customer churn
8%: previous customer churn
$43: user acquisition cost
78%: product adoption rate
87: employee satisfaction points
34%: operating margin

Step 2:
92%: customer satisfaction
45%: revenue growth
23%: market share
5%: customer churn
8%: previous customer churn
43.0: user acquisition cost
78%: product adoption rate
87%: employee satisfaction
34%: operating margin

Step 3:
Here are the lines sorted in descending order by numerical value:

92%: customer satisfaction
87%: employee satisfaction
78%: product adoption rate
45%: revenue growth
43.0: user acquisition cost
34%: operating margin
23%: market share
8%: previous customer churn
5%: customer churn

Step 4:
| Metric | Value |
|:--|--:|
| Customer Satisfaction | 92% |
| Employee Satisfaction | 87% |
| Product Adoption Rate | 78% |
| Revenue Growth | 45% |
| User Acquisition Cost | 43.0 |
| Operating Margin | 34% |
| Market Share | 23% |
| Previous Customer Churn | 8% |
| Customer Churn | 5% |
```

### 示例 2：用于利益相关方影响分析的并行化工作流

并发处理多个利益相关方群体的影响分析。

```python
# 示例 2：用于利益相关方影响分析的并行化工作流
# 并发处理多个利益相关方群体的影响分析

stakeholders = [
    """Customers:
    - Price sensitive
    - Want better tech
    - Environmental concerns""",
    """Employees:
    - Job security worries
    - Need new skills
    - Want clear direction""",
    """Investors:
    - Expect growth
    - Want cost control
    - Risk concerns""",
    """Suppliers:
    - Capacity constraints
    - Price pressures
    - Tech transitions""",
]

impact_results = parallel(
    """Analyze how market changes will impact this stakeholder group.
    Provide specific impacts and recommended actions.
    Format with clear sections and priorities.""",
    stakeholders,
)

for result in impact_results:
    print(result)
    print("+" * 80)
```

输出（节选；完整输出包含对客户、员工、投资者和供应商的分析）：

```
MARKET IMPACT ANALYSIS FOR CUSTOMERS
==================================

HIGH PRIORITY IMPACTS
-------------------
1. Price Sensitivity
- Rising inflation and costs likely to reduce purchasing power
- Increased competition for value-oriented products
- Risk of trading down to lower-cost alternatives

Recommended Actions:
• Introduce tiered pricing options
• Develop value-focused product lines
• Create loyalty programs with price benefits
• Highlight total cost of ownership benefits

2. Technology Demands
- Accelerating tech advancement creating higher expectations
- Integration of AI and smart features becoming standard
- Mobile/digital-first experience requirements

Recommended Actions:
• Accelerate digital transformation initiatives
• Invest in user experience improvements
• Develop smart product features
• Provide tech education and support

MEDIUM PRIORITY IMPACTS
----------------------
3. Environmental Consciousness
- Growing demand for sustainable products
- Increased scrutiny of environmental practices
- Willingness to pay premium for eco-friendly options

Recommended Actions:
• Develop eco-friendly product lines
• Improve packaging sustainability
• Communicate environmental initiatives
• Create recycling programs

MONITORING & METRICS
-------------------
• Track customer satisfaction scores
• Monitor price sensitivity metrics
• Measure adoption of new technologies
• Track sustainability-related purchases
• Regular customer feedback surveys

RISK FACTORS
------------
• Economic downturn impact on spending
• Tech adoption learning curve
• Cost vs. sustainability trade-offs
• Competition from specialized providers

TIMELINE PRIORITIES
------------------
Immediate (0-3 months):
- Price optimization
- Digital experience improvements

Short-term (3-12 months):
- Tech feature development
- Sustainability initiatives

Long-term (12+ months):
- Advanced technology integration
- Comprehensive eco-friendly transformation
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
[... 随后是员工、投资者、供应商的分析 ...]
```

### 示例 3：用于客户支持工单处理的路由工作流

基于内容分析将支持工单路由到合适的团队。

```python
# 示例 3：用于客户支持工单处理的路由工作流
# 基于内容分析将支持工单路由到合适的团队

support_routes = {
    "billing": """You are a billing support specialist. Follow these guidelines:
    1. Always start with "Billing Support Response:"
    2. First acknowledge the specific billing issue
    3. Explain any charges or discrepancies clearly
    4. List concrete next steps with timeline
    5. End with payment options if relevant

    Keep responses professional but friendly.

    Input: """,
    "technical": """You are a technical support engineer. Follow these guidelines:
    1. Always start with "Technical Support Response:"
    2. List exact steps to resolve the issue
    3. Include system requirements if relevant
    4. Provide workarounds for common problems
    5. End with escalation path if needed

    Use clear, numbered steps and technical details.

    Input: """,
    "account": """You are an account security specialist. Follow these guidelines:
    1. Always start with "Account Support Response:"
    2. Prioritize account security and verification
    3. Provide clear steps for account recovery/changes
    4. Include security tips and warnings
    5. Set clear expectations for resolution time

    Maintain a serious, security-focused tone.

    Input: """,
    "product": """You are a product specialist. Follow these guidelines:
    1. Always start with "Product Support Response:"
    2. Focus on feature education and best practices
    3. Include specific examples of usage
    4. Link to relevant documentation sections
    5. Suggest related features that might help

    Be educational and encouraging in tone.

    Input: """,
}

# 使用不同的支持工单进行测试
tickets = [
    """Subject: Can't access my account
    Message: Hi, I've been trying to log in for the past hour but keep getting an 'invalid password' error.
    I'm sure I'm using the right password. Can you help me regain access? This is urgent as I need to
    submit a report by end of day.
    - John""",
    """Subject: Unexpected charge on my card
    Message: Hello, I just noticed a charge of $49.99 on my credit card from your company, but I thought
    I was on the $29.99 plan. Can you explain this charge and adjust it if it's a mistake?
    Thanks,
    Sarah""",
    """Subject: How to export data?
    Message: I need to export all my project data to Excel. I've looked through the docs but can't
    figure out how to do a bulk export. Is this possible? If so, could you walk me through the steps?
    Best regards,
    Mike""",
]

print("Processing support tickets...\n")
for i, ticket in enumerate(tickets, 1):
    print(f"\nTicket {i}:")
    print("-" * 40)
    print(ticket)
    print("\nResponse:")
    print("-" * 40)
    response = route(ticket, support_routes)
    print(response)
    print("+" * 80)
```

输出：

```
Processing support tickets...


Ticket 1:
----------------------------------------
Subject: Can't access my account
    Message: Hi, I've been trying to log in for the past hour but keep getting an 'invalid password' error. 
    I'm sure I'm using the right password. Can you help me regain access? This is urgent as I need to 
    submit a report by end of day.
    - John

Response:
----------------------------------------

Available routes: ['billing', 'technical', 'account', 'product']
Routing Analysis:

This issue is clearly related to account access and authentication problems. The user is experiencing login difficulties with their password, which is a core account security and access issue. While there might be technical aspects involved, the primary concern is account access restoration. The urgency mentioned by the user and the nature of the problem (password/login issues) makes this a typical account support case. Account team specialists are best equipped to handle password resets, account verification, and access restoration procedures.


Selected route: account
Account Support Response:

Dear John,

I understand your urgency regarding account access. Before proceeding with account recovery, we must verify your identity to maintain security protocols.

Immediate Steps for Account Recovery:
1. Visit our secure password reset page at [secure portal URL]
2. Click "Forgot Password"
3. Enter your email address associated with the account
4. Follow the verification instructions sent to your email

Important Security Notes:
• The reset link expires in 30 minutes
• Do not share reset links or verification codes with anyone
• Ensure you're on our official website (check for https:// and correct domain)

WARNING: If you're unable to access your email or receive the reset link, additional verification will be required through our identity verification process.

Additional Security Recommendations:
- Enable Two-Factor Authentication after regaining access
- Review recent account activity for unauthorized access
- Update passwords on other accounts using similar credentials

Expected Resolution Time:
• Password reset: 5-10 minutes
• Identity verification (if needed): 1-2 business hours

If you continue experiencing issues, please respond with:
1. Account email address
2. Last successful login date
3. Any recent account changes

For urgent report submission, please contact your supervisor about deadline extension while we secure your account.

Regards,
Account Security Team
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Ticket 2:
----------------------------------------
Subject: Unexpected charge on my card
    Message: Hello, I just noticed a charge of $49.99 on my credit card from your company, but I thought
    I was on the $29.99 plan. Can you explain this charge and adjust it if it's a mistake?
    Thanks,
    Sarah

Response:
----------------------------------------

Available routes: ['billing', 'technical', 'account', 'product']
Routing Analysis:

This is clearly a billing-related inquiry as it involves:
1. Questions about charges on a credit card
2. Pricing plan discrepancy ($49.99 vs $29.99)
3. Request for explanation of charges
4. Potential request for refund/adjustment
The customer is specifically questioning a financial transaction, which falls under the billing team's expertise. While it touches on account plans, the core issue is about charges and payment processing.


Selected route: billing
Billing Support Response:

Dear Sarah,

I understand your concern about the unexpected charge of $49.99 when you were expecting to be billed $29.99.

After reviewing the charge, this difference typically occurs when:
1. The promotional period for the $29.99 rate has ended
2. There was a plan upgrade or change in service tier
3. Additional services were added to the account

To resolve this, I will:
1. Review your account history within 1 business day
2. Send you a detailed breakdown of the charges by email
3. If an error is confirmed, process a refund within 2-3 business days
4. Ensure your plan is set to the correct rate moving forward

For immediate action, you can:
- Review your current plan settings in your account dashboard
- Send us your last billing statement for comparison
- Provide any promotional codes you may have applied previously

Payment options available:
- If a refund is due, it will be credited to your original payment method
- You can adjust your plan back to $29.99 through your account settings if eligible
- Monthly auto-pay can be reviewed and updated as needed

Please let me know if you need any additional information or have questions about these next steps.

Best regards,
Billing Support Team
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Ticket 3:
----------------------------------------
Subject: How to export data?
    Message: I need to export all my project data to Excel. I've looked through the docs but can't
    figure out how to do a bulk export. Is this possible? If so, could you walk me through the steps?
    Best regards,
    Mike

Response:
----------------------------------------

Available routes: ['billing', 'technical', 'account', 'product']
Routing Analysis:

This is clearly a technical/how-to question about product functionality. The user is asking for specific instructions about a feature (data export), and mentions looking through documentation. The question is about product usage and requires technical knowledge to explain the export process. Keywords like "export," "data," "bulk export," and reference to documentation strongly indicate this is a technical support matter rather than billing, account, or general product inquiry.


Selected route: technical
Technical Support Response:

I'll help you export your project data to Excel. Here's the complete process:

Steps to Export Data:
1. Log into your project dashboard
2. Navigate to "Project Settings" in the top right corner
3. Select "Data Management" from the dropdown menu
4. Click the "Export" tab
5. Choose "Bulk Export" option
6. Select data range and specific fields to export
7. Choose "Excel (.xlsx)" as the output format
8. Click "Generate Export"
9. Wait for the system to process (may take 1-15 minutes depending on data size)
10. Download the exported file when ready

System Requirements:
- Supported browsers: Chrome 90+, Firefox 88+, Edge 91+
- Minimum 2GB RAM available
- Stable internet connection
- Excel 2016 or later for opening exported files

Common Issues & Workarounds:
A. If export times out:
   - Break data into smaller date ranges
   - Export during off-peak hours
   - Use filters to reduce data size

B. If download fails:
   - Clear browser cache
   - Use incognito/private window
   - Try a different supported browser

Escalation Path:
If you continue experiencing issues:
1. Contact your project administrator
2. Submit a ticket to technical support at [support@example.com]
3. Include your project ID and any error messages received
4. For urgent matters, call our support hotline: 1-800-XXX-XXXX
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
```

***

_来源：_[_Basic workflows | Claude Cookbook_](https://platform.claude.com/cookbook/patterns-agents-basic-workflows)
