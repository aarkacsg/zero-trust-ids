# Zero Trust IDS — Machine Learning Assisted Zero Trust Security Framework

## Overview

This project presents a simulation-based Zero Trust security framework that combines machine learning-based intrusion detection with contextual access-control decisions.

A Random Forest classifier trained using the NSL-KDD intrusion detection dataset estimates the probability that network traffic is malicious. This prediction is then passed to a simulated Zero Trust Policy Decision Point (PDP), where it is combined with contextual security attributes such as:

- Role-Based Access Control (RBAC)
- Device compliance
- Multi-Factor Authentication (MFA)
- Logical network segmentation
- Access context
- Machine learning attack probability

The Policy Decision Point evaluates the combined risk and generates one of three access decisions:

**Permit → Step-up Authentication → Deny**

This project was implemented as a controlled proof-of-concept simulation and was not deployed within a live enterprise network.

---

## Project Objective

The project investigated whether combining machine learning-based intrusion detection with contextual Zero Trust policy enforcement could improve attack containment compared with a simplified traditional perimeter-based security model.

The implementation focused on connecting intrusion-detection results directly to access-control decisions rather than treating the IDS as an independent security mechanism.

---

## Architecture

The framework follows the workflow below:

```
NSL-KDD Dataset
       │
       ▼
Data Preprocessing
       │
       ▼
Random Forest IDS
       │
       ▼
Attack Probability
       │
       ├─────────────────────┐
       ▼                     │
Zero Trust Policy Layer      │
       │                     │
       ├── User Role / RBAC  │
       ├── Device Compliance │
       ├── MFA Status        │
       ├── Network Segment   │
       └── Access Context    │
       │                     │
       ▼                     │
Composite Risk Score ◄───────┘
       │
       ▼
Policy Decision
       │
 ┌─────┼───────────────┐
 ▼     ▼               ▼
Permit Step-up Auth    Deny
```

### System Architecture

![Zero Trust System Architecture](images/01-system-architecture.png)

---

## Technologies Used

- Python 3
- Ubuntu Linux
- VMware Workstation
- Scikit-learn
- Pandas
- NumPy
- Joblib
- Matplotlib
- NSL-KDD dataset
- Random Forest
- Zero Trust Architecture concepts
- Role-Based Access Control
- Multi-Factor Authentication
- Logical micro-segmentation
- Contextual risk scoring

---

## Dataset

The Random Forest intrusion-detection component was trained and evaluated using the **NSL-KDD** dataset.

The implementation used the official training and testing files:

```
KDDTrain+.txt
KDDTest+.txt
```

Each record contains network traffic features together with an attack label.

The preprocessing workflow included:

- Converting attack labels into binary normal/attack classes
- Encoding categorical features
- Normalising numerical features
- Maintaining separate training and testing datasets
- Fitting preprocessing parameters only on the training data to reduce data leakage

---

## Random Forest Intrusion Detection

The Random Forest classifier was used to distinguish between normal and malicious network traffic.

The model generated:

1. A predicted traffic class
2. An attack probability

The attack probability was particularly important because it was not used as the final access decision.

Instead, it became one input to the Zero Trust Policy Decision Point.

### Random Forest Performance

| Metric | Result |
|---|---:|
| Accuracy | 78.21% |
| Precision | 96.67% |
| Recall | 63.93% |
| F1-score | 76.96% |
| False Positive Rate | 2.91% |
| False Negative Rate | 36.07% |
| Mean Detection Processing Time | 9.09 × 10⁻⁶ s |
| Training Time | 5.40 s |

The classifier achieved high precision, while the lower recall and higher false-negative rate demonstrated why intrusion detection alone should not be treated as the complete security decision mechanism.

The Zero Trust framework therefore evaluated additional contextual information before deciding whether access should be granted.

---

## Zero Trust Policy Decision Point

The simulated Policy Decision Point combined the Random Forest attack probability with contextual enterprise security information.

The evaluated attributes included:

| Attribute | Role in the framework |
|---|---|
| User Role / RBAC | Checks whether the requested resource matches authorised responsibilities |
| Device Compliance | Evaluates the security state of the requesting device |
| MFA Status | Provides additional identity-verification assurance |
| Network Segment | Identifies and controls cross-segment access |
| Access Context | Evaluates whether the activity matches expected conditions |
| Attack Probability | Provides the ML-derived indication of potentially malicious traffic |

These inputs contributed to a composite risk score.

The resulting policy logic was:

```
Low Risk      → Permit
Moderate Risk → Step-up Authentication
High Risk     → Deny
```

This enabled the framework to make adaptive decisions instead of relying on a simple trusted/untrusted model.

---

## Enterprise Scenarios

Five representative enterprise access scenarios were simulated:

1. **Normal Employee Access**  
   Legitimate access to an authorised resource under expected conditions.

2. **Unusual Legitimate Access**  
   Legitimate activity occurring under unusual contextual conditions.

3. **Unauthorised Resource Access**  
   An authenticated user attempting to access a resource outside the permissions associated with the assigned role.

4. **Compromised Device Access**  
   An access request originating from a device that does not satisfy expected security requirements.

5. **Lateral Movement Attempt**  
   Suspicious cross-segment access used to evaluate logical segmentation and lateral-movement containment.

---

## Policy Decision Distribution

The Policy Decision Point evaluated **22,544 simulated access requests**.

The resulting decisions were:

- **Permit:** 8,821
- **Step-up Authentication:** 5,417
- **Deny:** 8,306

This demonstrates that the framework did not rely solely on a binary allow-or-deny decision.

Intermediate-risk requests could instead require additional authentication.

![Zero Trust Policy Decision Distribution](images/02-policy-decision-distribution.png)

---

## Operational Security Results

The Zero Trust framework was evaluated using operational security metrics in addition to conventional machine learning metrics.

Key results included:

| Metric | Result |
|---|---:|
| Overall Policy Conformance | 81.09% |
| Normal Access Permit Rate | 97.63% |
| Unusual Legitimate Acceptance Rate | 97.46% |
| Unauthorised Access Denial Rate | 100% |
| Controlled Attack Containment | 78.31% |
| Controlled Lateral-Movement Containment | 43.12% |
| Controlled Compromised-Device Containment | 100% |

These measurements evaluate what happened after intrusion-detection evidence was incorporated into access-control decisions.

---

## Traditional Perimeter vs Zero Trust

A simplified traditional perimeter-based model was implemented as a controlled baseline.

Unlike the Zero Trust framework, the baseline did not continuously evaluate contextual attributes such as:

- User role
- Device compliance
- MFA status
- Network segmentation
- Access context

### Security Effectiveness Comparison

| Metric | Traditional Baseline | Zero Trust |
|---|---:|---:|
| Attack Containment | 53.90% | **78.31%** |
| Malicious Traffic Prevention | 53.90% | **78.31%** |
| Scenario-Driven Lateral-Movement Containment | 32.29% | **43.12%** |
| Compromised-Device Containment | 31.96% | **100%** |
| False Negative Rate | 46.10% | **19.57%** |

Attack containment improved by **24.41 percentage points** under the Zero Trust framework compared with the simplified traditional baseline.

![Traditional Perimeter vs Zero Trust Security Effectiveness](images/03-security-effectiveness-comparison.png)

---

## Policy Conformance Across Scenarios

Policy conformance was also evaluated across the five simulated enterprise-access scenarios.

The results demonstrate that the framework behaved differently depending on contextual conditions rather than applying the same security decision to every request.

Higher-risk scenarios involving compromised devices, unauthorised resources and lateral movement resulted in stricter access-control decisions.

![Zero Trust Policy Conformance](images/04-policy-conformance.png)

---

## Key Findings

The experiment demonstrated several important security concepts:

- Intrusion-detection accuracy alone does not determine whether enterprise resources are protected.
- Machine learning can provide a useful risk signal without becoming the sole security decision mechanism.
- Context such as identity, device state, MFA and network location can materially affect access decisions.
- Step-up authentication provides an intermediate response between allowing and denying access.
- Logical segmentation and contextual policy enforcement can improve containment of malicious activity.
- Operational security metrics such as attack containment and policy conformance provide additional insight beyond classifier accuracy alone.

---

## SOC Relevance

Although this project focuses on Zero Trust Architecture rather than a traditional SIEM workflow, several concepts are relevant to Security Operations:

- Correlating multiple security signals before making a decision
- Distinguishing detection from response
- Risk-based decision making
- Identity and access monitoring
- Device trust evaluation
- Lateral-movement containment
- False-positive and false-negative analysis
- Context-aware threat assessment
- Continuous verification
- Evidence-based security decisions

A SOC analyst should not rely on one alert, model result or telemetry source in isolation. Security decisions become stronger when multiple signals are correlated with user, device and network context.

---

## Limitations

This project was conducted as a controlled simulation.

It did not include:

- A live enterprise network
- Production identity-management systems
- Commercial Zero Trust platforms
- Physical network segmentation
- Live SIEM integration
- Real-time threat-intelligence feeds
- Production-scale user populations

The NSL-KDD dataset is also an older benchmark dataset and does not fully represent modern cloud, zero-day or advanced persistent threat activity.

The results should therefore be interpreted as demonstrating the feasibility and behaviour of the proposed framework rather than production performance.

---

## Future Improvements

Potential extensions include:

- Evaluating newer intrusion-detection datasets
- Comparing Random Forest with additional machine learning models
- Incorporating user behavioural analytics
- Adding continuous device-health telemetry
- Integrating threat-intelligence feeds
- Connecting the policy framework with an enterprise IAM platform
- Integrating with Microsoft Sentinel or another SIEM
- Testing in hybrid-cloud environments
- Evaluating production-scale access requests
- Strengthening lateral-movement policies
- Incorporating automated response and policy enforcement

---

## Project Scope

This repository documents my Master's cybersecurity project:

**Implementing and Assessing Zero Trust Security Models in Enterprise Networks**

The implementation is a **simulation-based proof of concept** designed to evaluate how machine learning-derived intrusion-detection signals can be incorporated into contextual Zero Trust access-control decisions.
