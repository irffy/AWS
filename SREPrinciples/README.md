# Service Level Indicators (SLIs), Objectives (SLOs), and Agreements (SLAs)

## 1️⃣ SLI (Service Level Indicator) – What are we measuring?
**SLI** is a performance metric that shows how well the service is performing.

### Example (Online Shopping Website):
- The SLI could be "Website uptime" (how often the site is available).
- If the website was online for 99.5% of the time in a month, then the SLI = 99.5%.


## 2️⃣ SLO (Service Level Objective) – What is our goal?
**SLO** is the internal target the company aims to meet to keep users happy.

### Example (Online Shopping Website):
- The company sets an SLO of 99.9% uptime (the site should be online 99.9% of the time).
- If actual uptime (SLI) is 99.5%, the team missed the SLO and should improve performance.

### Purpose:
- SLO helps teams work towards a higher standard than what they legally promise (SLA).

## 3️⃣ SLA (Service Level Agreement) – What is promised to customers?
**SLA** is a contract with customers. If the service fails, the company must compensate users.

### Example (Online Shopping Website):
- The company guarantees an SLA of 99% uptime.
- If uptime drops below 99%, the company must offer refunds or discounts to customers.
- If actual uptime (SLI) is 98.5%, the SLA is violated, and the company owes compensation.

## 📌 Summary Table

| Term                        | Definition                              | Example (Online Shopping Site)            |
|-----------------------------|-----------------------------------------|-------------------------------------------|
| **SLI (Service Level Indicator)** | A metric that measures performance    | Website uptime (e.g., 99.5%)              |
| **SLO (Service Level Objective)** | A target goal for performance          | Aim for 99.9% uptime                      |
| **SLA (Service Level Agreement)** | A legal commitment to customers        | Guarantee 99% uptime or give refunds      |

## 📉 Bad Performance Case:
- **SLI** = 98.5% uptime
- **SLO** (99.9%) not met → Needs improvement
- **SLA** (99%) not met → Customers get compensation

## 📈 Good Performance Case:
- **SLI** = 99.6% uptime
- **SLO** (99.9%) not met, but **SLA** (99%) met → No customer refunds needed

## 🎯 Simple Analogy
Imagine a pizza delivery service:
- 🍕 **SLI** = % of pizzas delivered within 30 minutes.
- 🎯 **SLO** = Internal target: Deliver 99% of pizzas on time.
- 📜 **SLA** = Promise to customers: If less than 95% are on time, they get a refund.