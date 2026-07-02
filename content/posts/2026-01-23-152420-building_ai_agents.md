---
title: "Building AI Agents"
date: 2026-01-23
draft: false
---

## single aggregation service {#single-aggregation-service}

-   AWS Bedrock
-   Google Vertex AI
-   Microsoft Foundry


## Topics {#topics}

Overview of LLMs and AI agent benchmark datasets

Running your own small LLMs on your laptop and Google CoLab

Agent control flows (HF Pipelines, HF smolagents, LangChain/LangGraph, Toolformer, Model Context Protocol)

Few-shot learning AKA in-context learning

Chain of thought reasoning (CoT) and self-refinement (Self-Refine)

Search-augmented generation AKA live RAG AKA internet-augmented dialog generation

Vector-database Retrieval-augmented generation (RAG)

Action Models (ReAct)

Vision-language models (VLMs)

Multi-agent systems (Generative Agents, ToolOrchestra, Magnetic-One)

Weight fine-tuning (QLORA)

Context Management (AgentFold)


## Papers {#papers}

-   Masterman, Tula, Sandi Besen, Mason Sawtell, and Alex Chao. "The Landscape of Emerging AI Agent Architectures for Reasoning, Planning, and Tool Calling: A Survey." arXiv preprint arXiv:2404.11584 (2024). <https://arxiv.org/abs/2404.11584>.
-   Training AI Co-Scientists Using Rubric Rewards.  Shashwat Goel, Rishi Hazra, Dulhan Jayalath, Timon Willi, Parag Jain, William F. Shen, Ilias Leontiadis, Francesco Barbieri, Yoram Bachrach, Jonas Geiping, Chenxi Whitehouse. <https://arxiv.org/abs/2512.23707>
-   QLoRA: Efficient Finetuning of Quantized LLMs, Tim Dettmers, Artidoro Pagnoni, Ari Holtzman, Luke Zettlemoyer, 2023. <https://arxiv.org/abs/2305.14314>
-   A generative model of memory construction and consolidation,  Eleanor Spens &amp; Neil Burges, 2023. <https://www.nature.com/articles/s41562-023-01799-z.pdf>
-   Magentic-One: A Generalist Multi-Agent System for Solving Complex Tasks. Microsoft Research AI Frontiers. 2024. <https://arxiv.org/html/2411.04468v1>
-   Generative Agents: Interactive Simulacra of Human Behavior. Joon Sung Park, Joseph C. O'Brien, Carrie J. Cai, Meredith Ringel Morris, Percy Liang, Michael S. Bernstein, 2023 <https://arxiv.org/abs/2304.03442>
-   Chain-of-Thought Prompting Elicits Reasoning in Large Language Models, Wei, J., et al. (2022). <https://arxiv:2201.11903>.
-   Internet-Augmented Dialogue Generation,  Komeili et al., 2021.  from Meta AI Research. This was one of the first papers to systematically explore augmenting conversational AI with real-time web search. <https://arxiv.org/abs/2107.07566>
-   Toolformer: Language Models Can Teach Themselves to Use Tools.  Timo Schick, Jane Dwivedi-Yu, Roberto Dessì, Roberta Raileanu, Maria Lomeli, Luke Zettlemoyer, Nicola Cancedda, Thomas Scialom, 2023. <https://arxiv.org/abs/2302.04761>
-   Language Models are Few-Shot Learners, Brown et al., 2020. <https://arxiv.org/abs/2005.14165>
-   A Survey on In-Context Learning, Dong, Q. et al. (2024). <https://arxiv.org/abs/2301.00234>
-   Formalizes ICL, relates it to meta-learning and prompting, and surveys techniques, analyses, and applications specifically for LLMs.
-   Long Is More for Alignment: A Simple but Tough-to-Beat Baseline for Instruction Fine-Tuning, 2024, Hao Zhao, Maksym Andriushchenko, Francesco Croce, Nicolas Flammarion. <https://arxiv.org/abs/2402.04833>
-   Mathematical exploration and discovery at scale, Bogdan Georgiev, Javier Gómez-Serrano, Terence Tao, Adam Zsolt Wagner, 2025. <https://arxiv.org/abs/2511.02864>
-   ToolOrchestra: Elevating Intelligence via Efficient Model and Tool Orchestration, 2025.  <https://arxiv.org/abs/2511.21689>
-   From Code Foundation Models to Agents and Applications: A Comprehensive Survey and Practical Guide to Code Intelligence, 2025. <https://arxiv.org/abs/2511.18538>
-   Small Language Models are the Future of Agentic AI. Peter Belcak, Greg Heinrich, Shizhe Diao, Yonggan Fu, Xin Don, Saurav Muralidhara, Yingyan Celine Lin, Pavlo Molchanov. <https://arxiv.org/pdf/2506.02153>
-   AgentFold: Long-Horizon Web Agents with Proactive Context Management. Rui Ye, Zhongwang Zhang, Kuan Li, Huifeng Yin, Zhengwei Tao, Yida Zhao, Liangcai Su, Liwen Zhang, Zile Qiao, Xinyu Wang, Pengjun Xie, Fei Huang, Siheng Chen, Jingren Zhou, Yong Jiang. <https://arxiv.org/abs/2510.24699>
-   ReAct: Synergizing Reasoning and Acting in Language Models. Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, Yuan Cao. <https://arxiv.org/abs/2210.03629>
-   Lean Copilot: Large Language Models as Copilots for Theorem Proving in Lean. Peiyang Song, Kaiyu Yang, Anima Anandkumar. <https://arxiv.org/abs/2404.12534>
-   Shunyu Yao, Dian Yu, Jeffrey Zhao, Izhak Shafran, Thomas L. Griffiths, Yuan Cao, and Karthik Narasimhan. 2023. Tree of Thoughts: Deliberate Problem Solving with Large Language Models. arXiv preprint arXiv:2305.10601. <https://arxiv.org/abs/2305.10601>
-   Aman Madaan, Niket Tandon, Prakhar Gupta, Skyler Hallinan, Luyu Gao, Sarah Wiegreffe, Uri Alon, Nouha Dziri, Shrimai Prabhumoye, Yiming Yang, Shashank Gupta, Bodhisattwa Prasad Majumder, Katherine Hermann, Sean Welleck, Amir Yazdanbakhsh, and Peter Clark. 2023. Self-Refine: Iterative Refinement with Self-Feedback. arXiv preprint arXiv:2303.17651. <https://arxiv.org/abs/2303.17651>
-   Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, and Yuan Cao. 2023. ReAct: Synergizing Reasoning and Acting in Language Models. arXiv preprint arXiv:2210.03629. <https://arxiv.org/abs/2210.03629>
-   Shukang Yin, Chaoyou Fu, Sirui Zhao, Ke Li, Xing Sun, Tong Xu, and Enhong Chen. 2024. A survey on multimodal large language models. Natl. Sci. Rev. 11, 12 (November 2024), nwae403. <https://doi.org/10.1093/nsr/nwae403>
-   Hanjia Lyu, Jinfa Huang, Daoan Zhang, Yongsheng Yu, Xinyi Mou, Jinsheng Pan, Zhengyuan Yang, Zhongyu Wei, Jiebo Luo. GPT-4V(ision) as A Social Media Analysis Engine. <https://arxiv.org/abs/2311.07547>
-   Jiacheng Miao, Joe R. Davis, Jonathan K. Pritchard, and James Zou. 2025. Paper2Agent: Reimagining Research Papers As Interactive and Reliable AI Agents. arXiv preprint arXiv:2509.06917. <https://arxiv.org/abs/2509.06917>
-   Model-First Reasoning LLM Agents: Reducing Hallucinations through Explicit Problem Modeling, Annu Rana, Gaurav Kumar, 2025. <https://arxiv.org/abs/2512.14474>
