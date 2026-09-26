---
title: "Internship Report & Technical Workshop"
date: 2026-08-03
weight: 1
chapter: false
---

# Graduation Internship Report & Advanced Technical Workshop

### Topic: Research on Amazon Web Services Cloud Computing and Serverless Architecture Implementation
**Capstone Project:** 100% Serverless Phishing Email Detection and Alert System on AWS Integrated with Machine Learning and Browser Client (Chrome Extension for Gmail).

---

### 1. Student Information & Academic Institution
* **Full Name:** Do Minh Vuong
* **Student ID (SID):** 0030268
* **Class:** 68CNMHT
* **Major:** Computer Systems and Network Engineering
* **Faculty:** Information Technology
* **University:** Hanoi University of Civil Engineering (HUCE)
* **Phone Number:** (+84) 987.654.321
* **Contact Email:** vuong.dmforwork@gmail.com
* **Academic Advisor (Lecturer):** M.Sc. Nguyen Viet Nhat — Faculty of Information Technology, HUCE

### 2. Internship Enterprise & Training Facility
* **Host Enterprise:** AMAZON WEB SERVICES VIETNAM COMPANY LIMITED (AWS Vietnam)
* **Legal Registered Address:** 36th Floor, Bitexco Financial Tower, No. 2 Hai Trieu Street, Ben Nghe Ward, District 1, Ho Chi Minh City
* **Actual Working & Training Location:** AWS Hanoi Office — 7th Floor, Grand Terra Building, 36 Cat Linh Street, O Cho Dua Ward, Dong Da District, Hanoi
* **Internship Position:** Cloud Solutions Architect & Systems Engineering Intern
* **Training Program:** First Cloud AI Journey (FCAJ Workforce Bootcamp 2026)
* **Mentor Lead:** Nguyen Gia Hung — Senior Solutions Architect, AWS Vietnam
* **Field Supervisor:** Pham Van Phong / Do Tuan Anh — Solutions Architect, AWS Vietnam
* **Internship Duration:** August 3, 2026 – September 27, 2026 (08 weeks of intensive training) plus a 04-week advanced roadmap (Weeks 9 – 12).

---

{{% notice info %}}
**Capstone Project Technical Summary (Phishing Email Detection on AWS):**  
* **Context & Problem:** Phishing attacks are evolving with unprecedented complexity, causing billions in corporate losses annually. Traditional security gateways rely on heavy, 24/7 dedicated servers (EC2/VMs) that incur high idle costs and lack real-time decentralized alerting.
* **Architecture Solution:** A real-time natural language classification pipeline based on **XGBoost + TruncatedSVD** (>97% accuracy), containerized with **Docker** and stored in **Amazon ECR**, executed serverlessly via **AWS Lambda**, exposed through **Amazon API Gateway v2 (HTTP API)** with strict CORS governance.
* **Audit Logging & Incident Alerting:** Every invocation is captured as an audit log in **Amazon DynamoDB** NoSQL database in real time. If an email is classified as dangerous phishing with confidence **≥ 90%**, an automated pub/sub trigger publishes to **Amazon SNS**, dispatching urgent security notices to administrators. The whole lifecycle is observed via **Amazon CloudWatch**.
* **Key Advantages:** Ultra-low warm latency (< 15 ms), **$0.00 operating cost** under **AWS Free Tier**, and elastic scalability from zero to thousands of concurrent requests.
{{% /notice %}}

---

### System Architecture Overview Diagram

![System Architecture Overview](/images/architecture/serverless-system-architecture.svg)

---

### Table of Contents

1. [**Worklog (12-Week Roadmap)**](1-worklog/)  
   *Detailed daily logs across 8 intensive weeks at AWS Hanoi Office and 4 projected advanced sprint weeks.*
2. [**Project Proposal & Solution Design**](2-proposal/)  
   *Problem definition, objectives, serverless architecture analysis, $0 budget estimation, and risk management.*
3. [**Events Participated**](3-eventparticipated/)  
   *Event recap and technical takeaway report for AWS Vietnam Community Meetup: "AI Revolution & Open Claw".*
4. [**Hands-on Technical Workshop**](4-workshop/)  
   *Comprehensive step-by-step hands-on guide across 8 technical modules: Docker, ECR, Lambda, API Gateway v2, DynamoDB, SNS, Chrome Extension, and CloudWatch.*
5. [**Self-evaluation**](5-self-evaluation/)  
   *Self-performance assessment across 8 standardized academic & corporate criteria.*
6. [**Sharing and Feedback**](6-feedback/)  
   *Experience reflection, program evaluation, and recommendations for future cohort improvement.*
