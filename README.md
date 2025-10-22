# AWS Bedrock Converse API - Learning Notebook

A comprehensive Jupyter notebook demonstrating Amazon Bedrock's Converse API through four progressive examples: basic text generation, RAG, conversational AI, and prompt templates. Created while completing the **AWS Generative AI for Developers** Coursera course.

[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![AWS Bedrock](https://img.shields.io/badge/AWS-Bedrock-orange.svg)](https://aws.amazon.com/bedrock/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## 🎯 What You'll Learn

This single notebook takes you from basic prompts to advanced GenAI patterns:

| Section | Concept                  | Model Used         | Difficulty        |
| ------- | ------------------------ | ------------------ | ----------------- |
| **1**   | Basic Text Generation    | Titan Text Express | ⭐ Beginner       |
| **2**   | RAG with Knowledge Base  | Claude 3 Haiku     | ⭐⭐ Intermediate |
| **3**   | Multi-Turn Conversations | Claude 3 Haiku     | ⭐⭐ Intermediate |
| **4**   | Prompt Templates         | Nova Micro         | ⭐⭐⭐ Advanced   |

## 🚀 Quick Start

### Prerequisites

- AWS Account with Bedrock access
- Python 3.8+
- Jupyter Notebook or JupyterLab
- AWS credentials configured

### Installation

```bash
# Clone the repository
git clone https://github.com/saadamir1/aws-bedrock-learning-notebook.git
cd aws-bedrock-learning-notebook

# Install dependencies
pip install -r requirements.txt

# Configure AWS credentials (if not already done)
aws configure

# Launch Jupyter
jupyter notebook bedrock_examples.ipynb
```

### Enable Required Models

Before running the notebook, enable these models in [AWS Bedrock Console](https://console.aws.amazon.com/bedrock/):

1. **Amazon Titan Text Express** (`amazon.titan-text-express-v1`)
2. **Anthropic Claude 3 Haiku** (`anthropic.claude-3-haiku-20240307-v1:0`)
3. **Amazon Nova Micro** (`us.amazon.nova-micro-v1:0`)

**How to enable**:

- Go to AWS Bedrock Console → **Model access** → **Manage model access**
- Check the boxes for the models above → **Request model access**
- Wait for approval (usually instant)

## 📚 Notebook Structure

### Section 1: Basic Text Generation 🔤

**Lines: ~20** | **Runtime: <1 sec**

Learn the fundamentals of the Converse API:

- Single-turn prompt/response pattern
- Using Amazon Titan Text Express
- Understanding response structure
- Basic JSON handling

```python
response = bedrock.converse(
    modelId="amazon.titan-text-express-v1",
    messages=[{"role": "user", "content": [{"text": "Your prompt"}]}]
)
```

**Output Example**: One-line DevOps summary

---

### Section 2: RAG with Knowledge Base 📖

**Lines: ~25** | **Runtime: 2-5 sec**

Implement Retrieval-Augmented Generation:

- Connect to S3-backed Knowledge Base
- Retrieve relevant document context
- Generate contextual answers with Claude
- Combine retrieval + generation

```python
response = agent_runtime.retrieve_and_generate(
    input={"text": "Your question"},
    retrieveAndGenerateConfiguration={
        "type": "KNOWLEDGE_BASE",
        "knowledgeBaseConfiguration": {...}
    }
)
```

**Output Example**: Answer based on your internal documentation

**⚠️ Setup Required**:

1. Create S3 bucket with documents
2. Create Knowledge Base in Bedrock console
3. Update `knowledgeBaseId` in notebook (currently: placeholder)

---

### Section 3: Multi-Turn Conversational Chatbot 💬

**Lines: ~100** | **Runtime: <2 sec**

Build a context-aware chatbot:

- Define system prompts (personality/role)
- Maintain conversation history
- Handle follow-up questions
- Configure inference parameters

**Key Concepts**:

- **System Prompt**: Defines chatbot personality
- **Message History**: Enables contextual responses
- **Inference Config**: Controls creativity (temperature, topP, maxTokens)

```python
messages = [
    {"role": "user", "content": [{"text": "First question"}]}
]
first_reply = ask_model(messages)

# Add to history
messages.append({"role": "assistant", "content": [{"text": first_reply}]})

# Ask follow-up - model remembers context!
messages.append({"role": "user", "content": [{"text": "Follow-up"}]})
```

**Output Example**:

- First: "3 rock songs"
- Second: "3 rock songs by UK artists" (remembers previous request!)

---

### Section 4: Prompt Templates 🎨

**Lines: ~150** | **Runtime: 2-3 sec**

Create reusable prompt patterns:

- Template creation with placeholders `{{variable}}`
- Variable substitution via `promptVariables`
- Template reuse across multiple inputs
- Production-ready prompt management

**The Magic**: Templates are like functions with parameters

```python
# Step 1: Create template (once)
bedrock_agent.create_prompt(
    name="My-Template",
    variants=[{
        "templateConfiguration": {
            "chat": {
                "messages": [{
                    "content": [{"text": "Process: {{input_variable}}"}]
                }],
                "inputVariables": [{"name": "input_variable"}]
            }
        }
    }]
)

# Step 2: Call template (many times)
response = bedrock.converse(
    modelId=prompt_arn,  # ← Use template ARN!
    promptVariables={"input_variable": {"text": "actual data"}}
)
```

**Output Example**: YES/NO screening for venue booking requests

---

## 💻 Running the Notebook

### Option 1: Run All Cells

```
Cell → Run All
```

Executes all examples sequentially (recommended for first run)

### Option 2: Run Specific Sections

Click on any section and press `Shift + Enter` to run individual examples

### Option 3: Run from Command Line

```bash
jupyter nbconvert --to notebook --execute bedrock_examples.ipynb
```

## 🔧 Configuration

### AWS Region

Change region in notebook if needed:

```python
bedrock = boto3.client("bedrock-runtime", region_name="us-west-2")  # Change here
```

**Supported Regions**: `us-east-1`, `us-west-2`, `eu-west-1`, `ap-northeast-1`, etc.

### Inference Parameters

Adjust model creativity/behavior:

```python
inferenceConfig = {
    "temperature": 0.7,   # 0=deterministic, 1=creative
    "topP": 0.9,          # Nucleus sampling threshold
    "maxTokens": 512      # Max response length
}
```

### Knowledge Base Setup (Section 2 Only)

1. Create S3 bucket: `my-bedrock-kb-bucket`
2. Upload documents (PDF, TXT, MD, etc.)
3. Create Knowledge Base in Bedrock console
4. Copy Knowledge Base ID
5. Update in notebook:

```python
"knowledgeBaseId": "YOUR_KB_ID_HERE",  # Replace this!
```

## 📊 Expected Costs

Approximate AWS costs per notebook run (all sections):

| Section                   | Tokens | Estimated Cost |
| ------------------------- | ------ | -------------- |
| Basic Text Gen            | ~40    | $0.00002       |
| RAG Query                 | ~500   | $0.0003        |
| Chatbot (2 turns)         | ~200   | $0.0001        |
| Prompt Template (2 calls) | ~600   | $0.0004        |
| **Total per run**         | ~1,340 | **~$0.001**    |

**Note**: Costs may vary based on:

- Input/output token counts
- Model selection (Claude costs more than Titan)
- Knowledge Base query complexity

💡 **Tip**: Start with free tier limits. AWS Bedrock offers free tier for first 2 months.

## 🐛 Troubleshooting

### `AccessDeniedException: Could not access model`

**Solution**: Enable model access in Bedrock console

```
AWS Console → Bedrock → Model access → Manage model access → Select models → Request access
```

### `ResourceNotFoundException: Knowledge base not found`

**Solution**: Update `knowledgeBaseId` in Section 2

- Create Knowledge Base in Bedrock console first
- Copy the KB ID (format: `XXXXXXXXXX`)
- Paste into notebook

### `ValidationException: The security token included in the request is expired`

**Solution**: Refresh AWS credentials

```bash
aws configure
# Or refresh temporary credentials
```

### `ModuleNotFoundError: No module named 'boto3'`

**Solution**: Install requirements

```bash
pip install -r requirements.txt
```

### `ConflictException: Prompt already exists` (Section 4)

**This is normal!** The code handles this automatically:

- First run: Creates template
- Subsequent runs: Reuses existing template
- No action needed

### Jupyter Kernel Crashes

**Solution**: Reduce `maxTokens` in inference config

```python
"maxTokens": 256  # Reduce from 512
```

## 🔒 Security Best Practices

### ✅ DO

- Use IAM roles with least privilege
- Store credentials in `~/.aws/credentials`
- Enable AWS CloudTrail logging
- Use environment variables for sensitive IDs
- Review AWS billing regularly

### ❌ DON'T

- Commit AWS credentials to Git (`.gitignore` prevents this)
- Share Knowledge Base IDs publicly
- Use root account credentials
- Hard-code secrets in notebook
- Expose model ARNs unnecessarily

### Included Security Files

- **`.gitignore`**: Prevents credential commits
- **`.env`** support (optional): Store KB IDs as environment variables

## 📖 Learning Path

### Recommended Order

1. ✅ Run Section 1 → Understand basic API
2. ✅ Read Section 3 code → See conversation flow
3. ✅ Run Section 3 → Experience multi-turn chat
4. ⏭️ Skip Section 2 initially (requires KB setup)
5. ✅ Run Section 4 → Learn advanced patterns
6. 🔄 Return to Section 2 after creating Knowledge Base

### Next Steps After This Notebook

- 🔗 **[AWS Bedrock Agents](#)** - Tool use and function calling
- 🔗 **[AWS Bedrock Guardrails](#)** - Content filtering and safety
- 🔗 **[AWS Bedrock Flows](#)** - Complex workflow orchestration
- 🔗 **[LangChain + Bedrock](#)** - Framework integration

## 🎓 Course Information

This notebook was created while completing:

- **Course**: AWS Generative AI for Developers
- **Platform**: Coursera
- **Topics Covered**: Bedrock Runtime API, RAG, Agents, Prompt Engineering

🔗 [View Course on Coursera](https://www.coursera.org/)

## 📚 Additional Resources

### Official Documentation

- [AWS Bedrock Documentation](https://docs.aws.amazon.com/bedrock/)
- [Converse API Reference](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_Converse.html)
- [Boto3 Bedrock Runtime](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/bedrock-runtime.html)
- [Knowledge Bases for Bedrock](https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base.html)

### Helpful Guides

- [Bedrock Pricing Calculator](https://aws.amazon.com/bedrock/pricing/)
- [Model Selection Guide](https://docs.aws.amazon.com/bedrock/latest/userguide/models-supported.html)
- [Prompt Engineering Best Practices](https://docs.anthropic.com/claude/docs/prompt-engineering)

### Community

- [AWS Bedrock GitHub Examples](https://github.com/aws-samples/amazon-bedrock-samples)
- [r/aws Reddit Community](https://reddit.com/r/aws)

## 🤝 Contributing

Found a bug or want to improve the notebook?

1. Open an issue describing the problem
2. Or submit a pull request with fixes
3. Suggestions for new examples welcome!

## 📄 License

MIT License - Free to use for learning and development.

See [LICENSE](LICENSE) file for details.

## 🌟 Acknowledgments

- AWS Bedrock team for the excellent API
- Coursera and AWS Training for the course
- Anthropic for Claude models
- Amazon for Titan and Nova models

---

## 💡 Tips for Success

1. **Start Simple**: Run Section 1 first to verify setup
2. **Read Comments**: The notebook has extensive inline explanations
3. **Experiment**: Change prompts, try different models
4. **Check Costs**: Monitor AWS billing console regularly
5. **Save Often**: Jupyter auto-saves, but manual save is good practice
6. **Version Control**: Commit changes to track your learning progress

---

**⭐ If this notebook helped you learn AWS Bedrock, consider starring the repo!**

**📧 Questions?** Open an issue or reach out on [LinkedIn/Twitter]

**🔗 Share your learnings** using hashtag: `#AWSBedrock` `#GenerativeAI`
