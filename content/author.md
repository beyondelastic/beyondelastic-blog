+++
date = '2025-01-30T16:11:50+01:00'
draft = false
title = 'Author'
showZenMode = false
showTableOfContents = false
+++

<style>
  /* Author page only: use the full content width instead of the narrow prose cap */
  .article-content,
  #single_header { max-width: none; }
  /* Keep certificate/badge galleries a sensible size at the wider width */
  .article-content .gallery img { max-width: 150px !important; }
  /* Blinking terminal cursor after the last command in the intro block */
  @keyframes term-blink { 50% { opacity: 0; } }
  .intro-term .term-cursor {
    display: inline-block;
    margin-left: 0.15rem;
    font-weight: 700;
    animation: term-blink 1.1s step-end infinite;
  }
</style>

<div style="display:flex; flex-wrap:wrap; align-items:center; gap:1.5rem; margin-bottom:1rem;">
<div class="intro-term" style="flex:1 1 320px; min-width:0;">

<div class="highlight"><pre class="chroma"><code class="language-console" data-lang="console"><span class="gp">$</span> whoami
<span class="go">alexander-ullah</span>
<span class="gp">$</span> kubectl get engineer alexander-ullah -o yaml<span class="term-cursor">▋</span></code></pre></div>

</div>
<img src="/images/alex.jpg" alt="Alexander Ullah" style="flex:0 0 150px; width:150px; height:150px; object-fit:cover; border-radius:9999px;" />
</div>

```yaml
apiVersion: beyondelastic.io/v1
kind: Engineer
metadata:
  name: Alexander Ullah
  title: Solution Engineer – Cloud & AI Apps
  company: Microsoft
  region: EMEA
  location: Germany
  labels:
    experience: "20y+"
    focus: cloud-native-and-ai
spec:
  about: >
    GenAI and agentic systems are my focus today. I'm a hands-on, customer-focused
    technologist who is passionate about AI agents, Kubernetes, cloud-native
    platforms and modern app architectures. I help customers modernize from legacy
    to cloud, and share what I learn by blogging and speaking.
  expertise:
    - GenAI & multi-agent systems
    - Kubernetes & containers
    - Platform engineering
    - GitOps & software supply chain security
    - Networking & security
    - Solution selling & public speaking
  currentRole:
    company: Microsoft
    team: Digital & Application Innovation
    period: 2025-01 → present
    highlights:
      - GenAI & agentic app strategies for strategic German customers
      - Agentic DevOps & AI coding assistants to boost developer productivity
      - Leads MVPs, PoCs, RFPs & hackathons across Azure AI, Azure App & GitHub
      - Speaker at Agentic AI & DevOps Day
  previousRole:
    company: VMware (Tanzu)
    title: Principal Specialist Solution Engineer & CTO Ambassador
    period: 2011 → 2024-12
    highlights:
      - App modernization (12-Factor, 5Rs) with the Tanzu portfolio across EMEA
      - CTO Ambassador bridging R&D and field teams (2023–2024)
      - Speaker at VMworld 2019–2021 & VMware Explore 2022–2023
      - vExpert 2020–2023 & vExpert App Modernization
    progression:
      - Senior Lead Solution Engineer, Tanzu (2022–2024)
      - Lead Solution Engineer, Kubernetes (2019–2022)
      - Staff Technical Account Manager (2017–2019)
      - ...
  certifications:
    - Azure AI Engineer Associate
    - Azure Administrator Associate
    - Azure AI Fundamentals
    - Certified Kubernetes Administrator (CKA)
    - Certified Kubernetes Application Developer (CKAD)
    - Certified Kubernetes Security Specialist (CKS)
    - VMware vExpert
    - ...
  awards:
    - VMware MVP Specialist SE, CEMEA (Q1 FY23)
    - VMware "At Our Best" – Elevate Award
    - VMware "At Our Best" – Achieve Award
    - ...
  links:
    linkedin: https://www.linkedin.com/in/alexander-ullah-a278a0b7/
```

You can also find me on [LinkedIn](https://www.linkedin.com/in/alexander-ullah-a278a0b7/).

Please be aware that all posts and opinions are my own, see [Disclaimer](https://beyondelastic.github.io/disclaimer)

{{< gallery >}}
  <img src="/images/ms-badge1.png" class="grid-w33" />
  <img src="/images/ms-badge2.png" class="grid-w33" />
  <img src="/images/ms-badge3.png" class="grid-w33" />
{{< /gallery >}}
{{< gallery >}}
  <img src="/images/cks.png" class="grid-w33" />
  <img src="/images/ckad.png" class="grid-w33" />
  <img src="/images/cka.png" class="grid-w33" />
{{< /gallery >}}
{{< gallery >}}
  <img src="/images/vexpert2025.png" class="grid-w33" />
  <img src="/images/vexpertstars.png" class="grid-w33" />
  <img src="/images/vexpertapp.png" class="grid-w33" />
{{< /gallery >}}

