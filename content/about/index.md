---
title: "关于"
layout: "page"
---

<div style="background:#0d0d0d; border:1px solid #00ff41; border-radius:12px; padding:32px 36px; margin-bottom:32px; font-family:'Courier New',Consolas,'Source Code Pro',monospace;">

  <div style="color:#00ff41; font-size:0.8rem; margin-bottom:18px; opacity:0.6;">
    $ whoami
  </div>

  <div id="terminal-text" style="color:#00ff41; font-size:1.05rem; line-height:2; min-height:6em;">
    <span id="typed-output"></span><span id="cursor" style="display:inline-block;width:10px;background:#00ff41;animation:blink 0.8s step-end infinite;">&nbsp;</span>
  </div>

  <div id="tags-row" style="display:none; margin-top:24px; display:flex; flex-wrap:wrap; gap:8px; opacity:0;">
    <span style="background:rgba(0,255,65,0.1); color:#00ff41; border:1px solid rgba(0,255,65,0.4); border-radius:4px; padding:3px 12px; font-size:0.78rem;">#大模型算法</span>
    <span style="background:rgba(0,255,65,0.1); color:#00ff41; border:1px solid rgba(0,255,65,0.4); border-radius:4px; padding:3px 12px; font-size:0.78rem;">#Agent</span>
    <span style="background:rgba(0,255,65,0.1); color:#00ff41; border:1px solid rgba(0,255,65,0.4); border-radius:4px; padding:3px 12px; font-size:0.78rem;">#数据挖掘</span>
    <span style="background:rgba(0,255,65,0.1); color:#00ff41; border:1px solid rgba(0,255,65,0.4); border-radius:4px; padding:3px 12px; font-size:0.78rem;">#Kaggle Expert</span>
  </div>

</div>

<style>
@keyframes blink {
  0%, 100% { opacity: 1; }
  50% { opacity: 0; }
}
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(6px); }
  to   { opacity: 1; transform: translateY(0); }
}
</style>

<script>
(function() {
  var lines = [
    "👋 你好，我是麻辣香郭 (malaxg)",
    "",
    "一名专注于大模型算法 (LLM Engineer)",
    "与数据挖掘的 AI 探索者。",
    "",
    "目前在腾讯负责大模型应用落地，",
    "热衷于将前沿 AI 技术转化为现实生产力。"
  ];

  var fullText = lines.join("\n");
  var output = document.getElementById("typed-output");
  var tagsRow = document.getElementById("tags-row");
  var i = 0;

  function type() {
    if (i < fullText.length) {
      var ch = fullText[i];
      if (ch === "\n") {
        output.innerHTML += "<br>";
      } else {
        output.innerHTML += ch;
      }
      i++;
      setTimeout(type, ch === "\n" ? 120 : 45);
    } else {
      // 打字完成，显示标签行
      document.getElementById("cursor").style.display = "none";
      tagsRow.style.display = "flex";
      tagsRow.style.animation = "fadeIn 0.6s ease forwards";
    }
  }

  // 页面加载后延迟 400ms 开始
  if (document.readyState === "loading") {
    document.addEventListener("DOMContentLoaded", function() { setTimeout(type, 400); });
  } else {
    setTimeout(type, 400);
  }
})();
</script>

---

## 💼 个人履历 (Experience)

### 🏢 实习经历
* **腾讯科技（深圳）有限公司** | **大模型应用算法实习生** `2025.01 ~ 至今`![](https://img.shields.io/badge/Tencent-腾讯-0052D9?style=flat-square&logo=tencent&logoColor=white)
    *  隶属于腾讯广告团队，负责大模型应用落地与算法优化。

### 🎓 教育背景
* **重庆邮电大学** | **计算机科学与技术 · 硕士** `2024.09 ~ 2027.06`![](https://img.shields.io/badge/CQUPT-重庆邮电大学-800000?style=flat-square&logo=cqupt&logoColor=white)
    * 研究方向为**多模态情感分析**，主修大模型微调、智能体架构及多模态研究。
* **重庆邮电大学** | **数据科学与大数据技术 · 学士** `2020.09 ~ 2024.06`![](https://img.shields.io/badge/CQUPT-重庆邮电大学-800000?style=flat-square&logo=cqupt&logoColor=white)
    * 专业成绩 **GPA 3.41/4 (Top 20%)**，具备扎实的数据结构与算法基础。

---

## 🚀 技术雷达 (Skills)

### 🧠 核心领域
* **LLM 微调与对齐：** 熟练掌握 SFT、LoRA、QLoRA 等微调技术；了解 PPO、DPO 及其变种算法；熟悉 Qwen、DeepSeek、Llama 等架构。
* **RAG 与 Agent：** 专注研究 Query 重写、Schema 链接、向量检索及基于 MCP 的多 Agent 协同流。
* **工程落地：** 熟练使用 `vLLM` 优化模型推理，具备 NVIDIA 高性能算力集群实战经验。

---

## 📝 学术成果 (Publications)

* **2026.05** | **Dual-Perspective Multimodal Sentiment Analysis with MoE Fusion: Representation Learning via Semantic Resonance and Divergence** — *ICASSP 2026* · [IEEE Xplore](https://ieeexplore.ieee.org/document/11462471)

---

## 🏆 竞赛成果 (Milestones)

> **Kaggle Expert (2银 2铜)** —— *“在数据世界里，用代码逼近最优解。”*

* **2026.01** | **2025腾讯广告算法大赛** —— **31/ 2800+**
* **2026.01** | **Kaggle NFL Big Data Bowl 2026** —— **银牌** (21/771)
* **2025.11** | **Kaggle Jigsaw Community Rules Classification** —— **银牌** (96/2445)
* **2025.06** | **Kaggle BirdCLEF-2025** —— **铜牌** (118/2025)
* **2024.11** | **第四届中国移动“梧桐杯”大数据创新大赛** —— **一等奖**
* **2022.10** | **重庆市首届人工智能创新大赛 (谣言核查)** —— **一等奖**


---

## 📫 与我联系 (Connect with Me)

* **GitHub:** [malaxg](https://github.com/malaxg)
* **Email:** guoyixian_01@163.com

---
<p align="center">欢迎技术交流与合作！ 🎯</p>
