# Sample Agents

A collection of experimental AI agents that demonstrate practical applications of AI-powered automation and analytics. These agents are designed to help teams accelerate their AI adoption journey and explore real-world business use cases.

## Purpose

This repository showcases example AI agents that can be:
- **Explored** to understand AI agent capabilities
- **Customized** and adapted to fit your specific use cases
- **Extended** with additional capabilities
- **Referenced** as examples for building your own agents

## Available Agents

### [FinOps Analyst for Chargeback](chargeback_agent/)

An AI agent specialized in Elastic Cloud cost intelligence and chargeback analytics. Helps finance teams, engineering managers, and FinOps professionals analyze Elasticsearch Service consumption and implement fair cost allocation.

**Key Features:**
- ECU (Elastic Compute Unit) consumption tracking across deployments
- Weighted blended cost calculation with configurable models
- Multi-dimensional cost attribution (deployment, tier, datastream)
- Trend analysis with daily, monthly, and custom period breakdowns
- Currency conversion and transparent cost breakdowns

**Use Cases:**
- Cloud cost optimization and showback/chargeback
- Resource consumption analysis across teams and projects
- FinOps reporting and budget planning
- Cost allocation for shared infrastructure

**Tech Stack:** Elasticsearch, Kibana Agent Builder, OpenAI-compatible APIs

[View full documentation →](chargeback_agent/README.md)

---

## Getting Started

Each agent includes:
- **Agent configuration** (`agent/` directory): Core agent definition and system prompts
- **Tool definitions** (`tools/` directory): Individual tool schemas and configurations
- **Documentation**: Detailed README with setup instructions and usage examples

### Quick Start

1. Navigate to the agent directory you're interested in
2. Review the agent's README for specific requirements
3. Follow the setup instructions to deploy in your environment
4. Customize the configuration to match your needs

## 🛠️ Agent Structure

Each agent follows a consistent structure:

```
agent_name/
├── README.md                 # Agent documentation
├── agent/                    # Agent configuration
│   └── agent_config.json    # Core agent definition
└── tools/                    # Tool definitions
    ├── tool_1.json
    ├── tool_2.json
    └── ...
```

## Contributing

These agents are shared to benefit the community. If you:
- **Build** an interesting agent
- **Enhance** an existing agent
- **Fix** bugs or improve documentation
- **Have** suggestions for new agents

Feel free to open an issue or submit a pull request!

## License

These agents are provided as examples and starting points. Please review and adapt them to your organization's security and compliance requirements before production deployment.

## Disclaimer

The agents in this repository are experimental and designed for learning and adaptation. They should be:
- Tested thoroughly in non-production environments first
- Customized to your specific organizational needs
- Reviewed for security and compliance requirements
- Monitored and maintained according to your operations standards

---

**Built by [johannesmahne](https://github.com/johannesmahne)** | Contributions welcome!
