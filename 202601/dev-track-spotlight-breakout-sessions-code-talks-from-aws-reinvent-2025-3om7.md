---
Title: DEV Track Spotlight: Breakout Sessions & Code Talks from AWS re:Invent 2025
Description: 
Author: Gunnar Grosch
Date: 2026-01-05T22:06:40.000Z
Robots: noindex,nofollow
Template: index
---
<p>Over the past weeks, I have had the privilege of diving deep into 27 incredible Breakout Sessions and Code Talks from the AWS re:Invent 2025 Developer Community (DEV) track. This series, DEV Track Spotlight, showcases the amazing work of AWS Heroes alongside speakers from AWS and Amazon, covering everything from multi-agent AI systems to serverless architectures, from Rust performance to legacy modernization.</p>

<p>The DEV track at re:Invent 2025 featured 60 unique sessions delivered by 93 speakers from the AWS Community, including AWS Heroes, AWS Community Builders, and AWS User Group Leaders, alongside speakers from AWS and Amazon. </p>

<p>These sessions covered cutting-edge topics including GenAI &amp; Agentic AI (multi-agent systems, Strands Agents SDK, Amazon Bedrock), Developer Tools (Kiro, Kiro CLI, Amazon Q Developer, AI-driven development), Security (AI agent security, container security, automated remediation), Infrastructure (serverless, containers, edge computing, observability), Modernization (legacy app transformation, CI/CD, feature flags), and Data (Amazon Aurora DSQL, real-time processing, vector databases).</p>

<p>Whether you attended re:Invent or are catching up remotely, this summary provides a roadmap to all 27 sessions I have covered. Each session offers unique value, from production-ready patterns to philosophical insights about what it means to be a builder in the age of AI.</p>

<h2>
  
  
  What You Can Learn from This Series
</h2>

<p>This series takes you deep into 27 sessions that represent the cutting edge of cloud development. Rather than surface-level overviews, each post explores real implementations, production patterns, and lessons learned from practitioners building with these technologies today.</p>

<p>You'll find sessions that challenge conventional thinking, like Ben Kehoe's exploration of why the builder's job is not to build. You'll see live coding demonstrations showing how tools like Kiro and MCP are changing development workflows. You'll learn from production experiences about serverless resilience, cost optimization, and security challenges.</p>

<p>The sessions range from highly technical deep dives (building MCP servers in Rust, distributed SQL with Amazon Aurora DSQL) to strategic discussions about organizational transformation and AI adoption. Whether you're looking to implement multi-agent systems tomorrow or understand where the industry is heading, these sessions offer practical value.</p>

<p>Most importantly, these aren't vendor pitches. These are practitioners sharing real code, real demos, and real learnings from building production systems. The AWS Heroes featured here are solving actual problems, and their insights reflect hard-won experience.</p>




<h2>
  
  
  All 27 Breakout Sessions &amp; Code Talks
</h2>

<p>Here's your complete guide to every breakout session and code talk in the series:</p>

<h3>
  
  
  Part 1: Building Production Agent Swarms - Mastering Industrial AI (DEV311)
</h3>

<p><strong>Speakers:</strong> Betty Zheng (Senior Developer Advocate, AWS) and Trista Pan (AWS Data Hero, Senior AI Engineer at Tetrate)</p>

<p>Learn how to build production-ready multi-agent systems with real examples from Tetrate, including customer support agents and autonomous troubleshooting systems. Covers Strands Agents SDK, multi-agent patterns, enterprise RAG, and production guardrails.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-building-production-agent-swarms-mastering-industrial-ai-dev311-41b0">Read the post</a></p>

<h3>
  
  
  Part 2: Anti-Money Laundering Multi-agent Orchestration with AWS Strands (DEV326)
</h3>

<p><strong>Speakers:</strong> Ayyanar Jeyakrishnan (AWS AI Hero, Executive Director at WellsFargo), Bhuvaneswari Subramani (AWS DevTools Hero, Chief Cloud Evangelist at Intuitive.Cloud), and Vivek Raja P S (AWS AI Hero, VP Product at Nexstem)</p>

<p>Discover how three AWS Heroes built a serverless AML system using eight specialized agents to reduce false positives from 95% to under 10%. Features Strands Agents SDK, Amazon Bedrock AgentCore, and regulatory compliance patterns for financial services.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-anti-money-laundering-multi-agent-orchestration-with-aws-strands-dev326-6p8">Read the post</a></p>

<h3>
  
  
  Part 3: Red Team vs Blue Team - Securing AI Agents (DEV317)
</h3>

<p><strong>Speakers:</strong> Brian H. Hough (AWS DevTools Hero, CEO &amp; Founder at Tech Stack Playbook) and Brian Tarbox (AWS Community Hero, Principal Solutions Architect at Caylent)</p>

<p>Learn how to secure AI agents in production through live attack demonstrations and comprehensive defense strategies. Covers Amazon Bedrock Guardrails, IAM enforcement, deterministic workflows, and real-world security vulnerabilities.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-red-team-vs-blue-team-securing-ai-agents-dev317-3mpe">Read the post</a></p>

<h3>
  
  
  Part 4: AI Native Development - Strategies and Impact across Amazon and AWS (DEV323)
</h3>

<p><strong>Speakers:</strong> James Hood (Principal Software Engineer, AWS) and Lilia Abaibourova (Senior Principal Engineer, Amazon Prime Video)</p>

<p>Explore how Amazon and AWS evolved from AI-assisted to AI Native development with grassroots adoption strategies, organizational transformation, and the open-source Strands Agent SOPs framework for scaling AI across large organizations.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-ai-native-development-strategies-and-impact-across-amazon-and-aws-dev323-593j">Read the post</a></p>

<h3>
  
  
  Part 5: Designing Local Generative AI Inference with AWS IoT Greengrass (DEV316)
</h3>

<p><strong>Speakers:</strong> Kohei "Max" Matsushita (AWS Community Hero, Tech Evangelist at Soracom)</p>

<p>Learn how to design and operate local generative AI inference architectures with live robotic arm demonstrations. Covers VLA models, edge deployment patterns, and practical approaches for updating AI models using AWS IoT Greengrass.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-designing-local-generative-ai-inference-with-aws-iot-greengrass-dev316-3kgk">Read the post</a></p>

<h3>
  
  
  Part 6: Unleash Rust's Potential on AWS (DEV307)
</h3>

<p><strong>Speakers:</strong> AJ Stuyvenberg (AWS Serverless Hero) and Darko Mesaroš (Principal Developer Advocate, AWS)</p>

<p>Discover how Rust's safety, performance, and reliability make it ideal for cloud applications on AWS. Features production Rust implementations from Amazon Aurora DSQL to serverless functions, with real-world performance benchmarks and practical tips.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-unleash-rusts-potential-on-aws-dev307-4e31">Read the post</a></p>

<h3>
  
  
  Part 7: Build real-time full-stack Generative AI applications (DEV310)
</h3>

<p><strong>Speakers:</strong> Salih Gueler (Senior Developer Advocate, AWS) and Matt Goldberg (Senior Developer Advocate, AWS)</p>

<p>Learn how to build production-ready full-stack Generative AI applications from scratch through live coding demonstrations. Features Kiro, AWS CDK deployments, Amazon Bedrock integration, and the new AWS MCP server for agent-driven development.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-build-real-time-full-stack-generative-ai-applications-dev310-fcd">Read the post</a></p>

<h3>
  
  
  Part 8: The Art of Embracing Failures in Serverless Architectures (DEV312)
</h3>

<p><strong>Speakers:</strong> Anahit Pogosova (AWS Data Hero, Lead Cloud Architect at F-Secure)</p>

<p>Learn how to build resilient serverless architectures with real-world examples of handling failures in distributed systems. Covers AWS Lambda, Amazon Kinesis, and practical strategies for timeouts, retries, and error handling.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-the-art-of-embracing-failures-in-serverless-architectures-dev312-16ff">Read the post</a></p>

<h3>
  
  
  Part 9: Modernize Legacy Applications with Agentic AI (DEV333)
</h3>

<p><strong>Speakers:</strong> Phil de Valence (Senior Solutions Architect, AWS) and Matt Lewis (AWS Data Hero, Chief AWS Architect at IBM)</p>

<p>Discover AI-powered modernization patterns that transform legacy monoliths into cloud-native services through seven live demonstrations of AWS Transform, Amazon Q Developer, and Kiro in action.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-modernize-legacy-applications-with-agentic-ai-dev333-4cfa">Read the post</a></p>

<h3>
  
  
  Part 10: Spec-driven development with Kiro (DEV314)
</h3>

<p><strong>Speakers:</strong> Erik Hanchett (Senior Developer Advocate, AWS) and Nikhil Swaminathan (Principal Product Manager, AWS)</p>

<p>Learn how Kiro implements spec-driven development to transform AI coding from vibe coding to structured workflows. Features clear documentation, property-based testing, agent hooks, and audit trails for production-ready applications.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-spec-driven-development-with-kiro-dev314-45e8">Read the post</a></p>

<h3>
  
  
  Part 11: Building .NET AI Applications with Semantic Kernel and Amazon Bedrock (DEV302)
</h3>

<p><strong>Speakers:</strong> AM Grobelny (Principal Developer Advocate, AWS) and Nicki Stone (Principal Engineer, Amazon)</p>

<p>Learn how to build .NET AI applications with Semantic Kernel and Microsoft Agent Framework on Amazon Bedrock. Features practical demos, Amazon Bedrock AgentCore infrastructure, and MCP server integration for production-ready agentic applications.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-building-net-ai-applications-with-semantic-kernel-and-amazon-bedrock-dev302-dj2">Read the post</a></p>

<h3>
  
  
  Part 12: Building Agentic AI - Amazon Nova Act and Strands Agents in Practice (DEV327)
</h3>

<p><strong>Speakers:</strong> Haowen Huang (Senior Developer Advocate, AWS) and Jacky Wu (Senior Solutions Architect, AWS)</p>

<p>Explore practical applications of Agentic AI through two real-world case studies featuring Amazon Nova Act and the Strands Agents framework. Learn how to build a Hong Kong weather agent and a mathematics teaching AI agent with comprehensive code demonstrations.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-building-agentic-ai-amazon-nova-act-and-strands-agents-in-practice-dev327-3hbj">Read the post</a></p>

<h3>
  
  
  Part 13: The Builder's Job Is Not to Build: A Mindset for Better Outcomes (DEV347)
</h3>

<p><strong>Speakers:</strong> Ben Kehoe (AWS Serverless Hero, Distinguished Engineer at Siemens)</p>

<p>Discover how shifting from an output-focused to an outcome-focused mindset can transform your approach to software development. Challenges conventional thinking about what it means to be a builder and shares practical strategies for solving problems more effectively.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-the-builders-job-is-not-to-build-a-mindset-for-better-outcomes-dev347-50b9">Read the post</a></p>

<h3>
  
  
  Part 14: Advanced feature flags: Faster releases and rapid recovery (DEV320)
</h3>

<p><strong>Speakers:</strong> Steve Rice (General Manager, AWS AppConfig &amp; Parameter Store) and Ben Shumpert (Senior SDE, AWS)</p>

<p>Learn how Amazon leverages feature flags at scale with AWS AppConfig. Features real production examples, multi-variant segmentation, operational flags for MTTR reduction, and deployment safety patterns you can implement immediately.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-advanced-feature-flags-faster-releases-and-rapid-recovery-dev320-4k5i">Read the post</a></p>

<h3>
  
  
  Part 15: Build modern applications with Amazon Aurora DSQL (DEV308)
</h3>

<p><strong>Speakers:</strong> Oleksii Ivanchenko (Solutions Architect, AWS) and Vadym Kazulkin (AWS Serverless Hero, Head of Development at ip.labs)</p>

<p>Learn how to build modern serverless applications with Amazon Aurora DSQL. Features deep dives into distributed architecture, performance characteristics, and real-world implementation patterns for scalable database solutions.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-build-modern-applications-with-amazon-aurora-dsql-dev308-4g5o">Read the post</a></p>

<h3>
  
  
  Part 16: AI agents at the edge: Build for offline, scale in cloud (DEV301)
</h3>

<p><strong>Speakers:</strong> Ana Cunha (Senior Developer Advocate, AWS) and David Victoria (AWS Community Hero, Senior Cloud Architect at Caylent)</p>

<p>Learn how to build AI agents that work fully offline using Strands Agents SDK and Ollama, then seamlessly scale to cloud capabilities with Amazon Bedrock. Demonstrates practical patterns for edge-deployed agents in manufacturing, field operations, and data sovereignty scenarios.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-ai-agents-at-the-edge-build-for-offline-scale-in-cloud-dev301-4i7b">Read the post</a></p>

<h3>
  
  
  Part 17: Build a Multi-Agent Role-Playing Game Master with Strands Agents (DEV330)
</h3>

<p><strong>Speakers:</strong> Olivier Leplus (Senior Developer Advocate, AWS) and Tiffany Souterre (Senior Developer Advocate, AWS)</p>

<p>Discover how to build a multi-agent AI Game Master using Strands Agents SDK with live coding demonstrations. Features MCP and A2A protocols, specialized agents for character management and rule checking, and practical examples of creating production-ready agentic workflows.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-build-a-multi-agent-role-playing-game-master-with-strands-agents-dev330-4ia1">Read the post</a></p>

<h3>
  
  
  Part 18: Control Humanoid Robots and Drones with Voice and Agentic AI (DEV313)
</h3>

<p><strong>Speakers:</strong> Haowen Huang (Senior Developer Advocate, AWS) and Cyrus Wong (AWS AI Hero, Senior Lecturer at Hong Kong Institute of Information Technology)</p>

<p>Learn how to build voice-controlled robotics systems using Amazon Nova Sonic and Agentic AI. Features real-world examples with humanoid robots and drones, serverless architecture, and MCP server implementation for minimizing latency.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-control-humanoid-robots-and-drones-with-voice-and-agentic-ai-dev313-500e">Read the post</a></p>

<h3>
  
  
  Part 19: Optimize AWS Costs: Developer Tools and Techniques (DEV318)
</h3>

<p><strong>Speakers:</strong> Steph Gooch (Senior Solution Architect Advocate, AWS) and Kenneth Attard (AWS Community Hero, Enterprise Architect at Betsson Group)</p>

<p>Learn practical cost optimization strategies with real-world examples from Betsson Group. Features AI-powered tools like Amazon Q Developer and Kiro CLI, and advanced techniques for networking, databases, and serverless cost optimization.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-optimize-aws-costs-developer-tools-and-techniques-dev318-1obh">Read the post</a></p>

<h3>
  
  
  Part 20: AI Agents for Databases: Discover, Recommend, Optimize (DEV315)
</h3>

<p><strong>Speakers:</strong> Namrata H Shah (AWS Community Hero, Managing Director at Nuveen) and Rob Koch (AWS Data Hero, Senior Principal at Slalom)</p>

<p>Learn how to implement AI agents that monitor and analyze AWS database services. Features practical examples of detecting performance bottlenecks, surfacing query pattern issues, and suggesting optimization strategies for production databases.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-ai-agents-for-databases-discover-recommend-optimize-dev315-n0f">Read the post</a></p>

<h3>
  
  
  Part 21: Supercharge DevOps with AI-driven observability (DEV304)
</h3>

<p><strong>Speakers:</strong> Elizabeth Fuentes Leone (Developer Advocate GenAI, AWS) and Rossana Suarez (AWS Container Hero, Engineer at Naranjax)</p>

<p>Learn how to transform DevOps with AI-powered observability using Amazon Bedrock and Strands Agents SDK. Features live demos of proactive deployment protection and intelligent incident prevention for modern SRE practices.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-supercharge-devops-with-ai-driven-observability-dev304-4em3">Read the post</a></p>

<h3>
  
  
  Part 22: How Amazon Teams Use AI Assistants to Accelerate Development (DEV403)
</h3>

<p><strong>Speakers:</strong> James Hood (Principal Software Engineer, AWS) and Maish Saidel-Keesing (Senior Developer Advocate, AWS)</p>

<p>Learn how Amazon developers achieve real productivity gains using AI-powered coding assistants through proven techniques like Prompt-Driven Development and Agent Scripts. Discover practical strategies for accelerating development workflows while maintaining code quality.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-how-amazon-teams-use-ai-assistants-to-accelerate-development-dev403-1bh4">Read the post</a></p>

<h3>
  
  
  Part 23: Building AI Agents with Kiro, MCP, and Amazon Bedrock AgentCore (DEV331)
</h3>

<p><strong>Speakers:</strong> Erik Hanchett (Senior Developer Advocate, AWS) and Du'An Lightfoot (Senior Developer Advocate, AWS)</p>

<p>Learn how to build production-ready AI agents using Kiro, Model Context Protocol (MCP), and Amazon Bedrock AgentCore. Features practical demonstrations of agent architecture, memory management, and deployment patterns for scalable AI systems.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-building-ai-agents-with-kiro-mcp-and-amazon-bedrock-agentcore-dev331-jf1">Read the post</a></p>

<h3>
  
  
  Part 24: Compile blazing-fast MCP servers in Rust (DEV405)
</h3>

<p><strong>Speakers:</strong> Darko Mesaros (Principal Developer Advocate, AWS)</p>

<p>Learn how to build production-ready MCP servers in Rust with live coding demonstrations. Features real-world examples with receipt printers and practical insights on creating standalone binaries without dependency management headaches.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-compile-blazing-fast-mcp-servers-in-rust-dev405-4inm">Read the post</a></p>

<h3>
  
  
  Part 25: Build multi-modal AI agents with Strands Agents and Amazon S3 Vectors (DEV332)
</h3>

<p><strong>Speakers:</strong> Joy Chakraborty (Senior Technical Program Manager, AWS) and Elizabeth Fuentes Leone (Developer Advocate, AWS)</p>

<p>Learn how to build production-ready multimodal AI agents using the open-source Strands Agents SDK and Amazon S3 Vectors. Demonstrates creating agents that analyze images, videos, and documents while maintaining personalized conversations through persistent memory at billion-vector scale.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-build-multi-modal-ai-agents-with-strands-agents-and-amazon-s3-vectors-dev332-4jp5">Read the post</a></p>

<h3>
  
  
  Part 26: Serverless Full-Stack in Action: AI-Driven Developer Experience (DEV309)
</h3>

<p><strong>Speakers:</strong> Gunnar Grosch (Principal Developer Advocate, AWS) and Shridhar Pandey (Principal Product Manager Serverless, AWS)</p>

<p>Learn how to build serverless full-stack applications using Kiro CLI and AI coding assistants. This hands-on session demonstrates building complete serverless backends, creating frontends, and implementing production-grade observability - all through natural language prompts.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-serverless-full-stack-in-action-ai-driven-developer-experience-dev309-3231">Read the post</a></p>

<h3>
  
  
  Part 27: Building Scalable, Self-Orchestrating AI Workflows with A2A and MCP (DEV415)
</h3>

<p><strong>Speakers:</strong> Gunnar Grosch (Principal Developer Advocate, AWS) and Allen Helton (AWS Serverless Hero, Ecosystem Engineer at Momento)</p>

<p>Learn how to build production-ready, self-orchestrating AI workflows using Agent-to-Agent (A2A) protocol and Model Context Protocol (MCP). Features real-world examples from a serverless logistics platform built with AWS Lambda, Amazon Bedrock, and Momento Cache.</p>

<p><a href="https://dev.to/aws/dev-track-spotlight-building-scalable-self-orchestrating-ai-workflows-with-a2a-and-mcp-dev415-5bkn">Read the post</a></p>




<h2>
  
  
  About This Series
</h2>

<p>This post is part of <strong>DEV Track Spotlight</strong>, a series highlighting the incredible sessions from the AWS re:Invent 2025 Developer Community (DEV) track.</p>

<p>The DEV track featured <strong>60 unique sessions</strong> delivered by <strong>93 speakers</strong> from the AWS Community - including AWS Heroes, AWS Community Builders, and AWS User Group Leaders - alongside speakers from AWS and Amazon. These sessions covered cutting-edge topics including:</p>

<ul>
<li>  🤖 <strong>GenAI &amp; Agentic AI</strong> - Multi-agent systems, Strands Agents SDK, Amazon Bedrock</li>
<li>  🛠️ <strong>Developer Tools</strong> - Kiro, Kiro CLI, Amazon Q Developer, AI-driven development</li>
<li>  🔒 <strong>Security</strong> - AI agent security, container security, automated remediation</li>
<li>  🏗️ <strong>Infrastructure</strong> - Serverless, containers, edge computing, observability</li>
<li>  ⚡ <strong>Modernization</strong> - Legacy app transformation, CI/CD, feature flags</li>
<li>  📊 <strong>Data</strong> - Amazon Aurora DSQL, real-time processing, vector databases</li>
</ul>

<p>Each post in this series dives deep into one session, sharing key insights, practical takeaways, and links to the full recordings. Whether you attended re:Invent or are catching up remotely, these sessions represent the best of our developer community sharing real code, real demos, and real learnings.</p>

<p><strong>Follow along</strong> as we spotlight these amazing sessions and celebrate the speakers who made the DEV track what it was!</p>

