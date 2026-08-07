# Cloud VM Disk Utilization Monitoring — AWS (Ansible + CloudWatch)

A secure, multi-account, scalable solution to monitor disk utilization across **all EC2 VMs** in an AWS Organization, detect low disk space early, and minimize downtime risk.

It leverages the **existing stack (Ansible)** for discovery + enrollment and adds only the **cloud-native services that clearly earn their place** (SSM, CloudWatch, OAM, SNS).

---

## 1. High-Level Architecture



*
```
                    +--------------------------------+
                    |      AWS Organizations         |
                    | (Multiple AWS Accounts)        |
                    +---------------+----------------+
                                    |
                           Cross-Account IAM Role
                                    |
                    +---------------v----------------+
                    |      Monitoring Account        |
                    | Ansible Control Node           |
                    | CloudWatch Dashboard + Alarms  |
                    | SNS Notifications              |
                    +---------------+----------------+
                                    |
                        AWS Systems Manager (SSM)
                                    |
        -------------------------------------------------------
        |                      |                      |
+---------------+     +---------------+     +---------------+
| AWS Account A |     | AWS Account B |     | AWS Account C |
| EC2 + SSM     |     | EC2 + SSM     |     | EC2 + SSM     |
| CloudWatch    |     | CloudWatch    |     | CloudWatch    |
| Agent         |     | Agent         |     | Agent         |
+-------+-------+     +-------+-------+     +-------+-------+
        +---------------------+---------------------+
                              |
                 CloudWatch Metrics (via OAM)
                              |
                   Central Dashboard & Alerts

```

---

## 2. Repository Structure

```
disk-monitoring/
├── ansible.cfg
├── requirements.yml
├── inventories/
│   └── aws_ec2.yml                 # EC2 dynamic inventory (tag-based discovery)
├── group_vars/
│   └── all.yml                     # SSM connection + thresholds (single source of truth)
├── playbooks/
│   ├── discover_instances.yml      # validate SSM reachability
│   ├── install_agent.yml           # install CloudWatch Agent
│   ├── configure_agent.yml         # deploy config + start agent
│   ├── verify_agent.yml            # verify + on-demand df snapshot
│   └── deploy.yml                  # runs all of the above, in order
├── roles/
│   ├── discovery/                  # SSM connectivity check
│   ├── cloudwatch_install/         # OS-aware agent install (from official AWS URL)
│   ├── cloudwatch_config/          # templated config + restart-on-change handler
│   └── monitoring/                 # verify agent + df fallback collector
├── templates/
│   └── cloudwatch-agent.json.j2    # readable copy of the agent config
├── cloudformation/
│   ├── cross-account-role.yaml         # access mgmt: role the hub assumes (StackSet)
│   ├── ec2-instance-profile.yaml       # access mgmt: badge on each VM (SSM + CW)
│   ├── monitoring-account-oam-sink.yaml# aggregation: central metrics sink
│   ├── member-account-oam-link.yaml    # aggregation: per-account link -> sink (StackSet)
│   └── disk-alarms.yaml                # alerting: SNS + single 80% disk-usage alarm
└── docs/
    ├── architecture.svg

```

---

## 3. The Four Requirements → Where They Live

### A. Data Collection

- **Primary:** `roles/cloudwatch_install` + `roles/cloudwatch_config` install and configure the CloudWatch Agent, which emits `disk_used_percent`, inodes, used, total **every 60s**, dimensioned by `InstanceId` + mount path.
- **Fallback / on-demand:** `roles/monitoring` runs `df` over SSM and publishes a custom metric.

### B. Data Aggregation

- **CloudWatch cross-account Observability (OAM):** each account creates a **Link** (`member-account-oam-link.yaml`) to a single **Sink** (`monitoring-account-oam-sink.yaml`) in the monitoring account → one dashboard, one set of alarms, **no ETL**.
- **Alerting:** `disk-alarms.yaml` — SNS topic + a single **80%** disk-usage alarm.

### C. Access Management

- **No SSH.** `group_vars/all.yml` sets `ansible_connection: aws_ssm`, so Ansible reaches every VM through **AWS Systems Manager** — no keys, no bastion, no port 22, fully audited via CloudTrail.
- **Cross-account:** the hub uses **STS AssumeRole** into `DiskMonitoringCrossAccountRole` (`cross-account-role.yaml`, least-privilege, `ExternalId`-scoped). VMs carry an instance profile (`ec2-instance-profile.yaml`) granting only SSM + CloudWatch agent permissions.

### D. Scalability

- **Discovery is automatic:** `inventories/aws_ec2.yml` returns every running VM tagged `Monitoring=Enabled` — no manual host lists.
- **Enrollment is org-wide & self-healing:** CloudFormation **StackSets** roll the IAM role, instance profile, and OAM link to every account (incl. new accounts via Organizations auto-deployment); **SSM State Manager** keeps the agent installed under drift.

---

---

## 5. Solution Workflow

```
Developer launches EC2  →  tag Monitoring=Enabled  →  aws_ec2 inventory discovers it
  →  ansible-playbook deploy.yml (discover → install → configure → verify)
  →  CloudWatch Agent collects disk metrics (60s)  →  metrics flow via OAM to the hub
  →  central dashboard updates  →  alarm at 80%  →  SNS → email / Slack

```

---

## 6. Key Components Summary

| Requirement | Solution |
| --- | --- |
| Data Collection | CloudWatch Agent installed & configured by Ansible (df fallback via `monitoring` role) |
| Aggregation | Central CloudWatch dashboard fed by cross-account **OAM** |
| Access Management | **IAM roles + AWS Systems Manager (SSM)**, no SSH keys, audited by CloudTrail |
| VM Discovery | **EC2 Dynamic Inventory** plugin (tag `Monitoring=Enabled`) |
| Enrollment | Automatic via Ansible playbooks/roles over SSM |
| Scalability | New EC2s auto-discovered & configured; StackSets seed new accounts |
| Alerting | CloudWatch Alarm (80%) + Amazon SNS |
| Security | IAM, SSM, CloudTrail, least privilege, no inbound ports |

### Access Management (detail)

| Component | Purpose |
| --- | --- |
| IAM Role | Secure, least-privilege permissions for EC2 and Ansible |
| AWS Systems Manager | Secure access without SSH |
| Cross-Account IAM Role | Access multiple AWS accounts from one hub |
| CloudTrail | Audit all actions |

### VM Discovery & Enrollment (detail)

- **Discovery:** Ansible queries the AWS EC2 API via the dynamic inventory plugin; any running instance tagged `Monitoring=Enabled` appears automatically — no IPs to maintain.
- **Enrollment:** on discovery, Ansible (over SSM) → installs the agent → configures monitoring → starts it → verifies status. The server then streams disk metrics to CloudWatch.

