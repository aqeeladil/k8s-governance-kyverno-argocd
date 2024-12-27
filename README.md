# Introduction to Kyverno

## Problem Statement

In large organizations, Kubernetes clusters have thousands of microservices and namespaces. Each team works on separate namespaces, but ensuring they follow organizational compliance rules (e.g., every pod must have resource limits) is challenging.

#### Common Problems Without Governance:
- Resource Exhaustion: Example: A single pod consumes excessive memory (50GB) on a node, causing other pods to fail.
- Image Tag Mismanagement: Using latest image tags leads to unpredictable behavior during pod restarts.
Security Risks: Lack of proper security configurations in pods

#### Traditional Solution:
- Kubernetes Admission Controllers
    - Written in Go language.
    - Complex and hard to scale.
    - Frequent rule changes require rewriting controllers.

#### Modern Solution:
- Kyverno
    - A Dynamic Admission Controller.
    - A Kubernetes-native policy engine
    - No need to write complex Go code.
    - Policies are written in YAML files.
    - Simplifies enforcing Kubernetes security and compliance rules.

## Core Features of Kyverno

- **Validate:** Enforce compliance rules. For example;
    - Ensure pods have resource limits.
    - Block users from using `latest` tag in the deployment or pod resources.
    - Namespaces must have quotas.
- **Mutate:** Modify incoming requests or resource definitions automatically. For example;
    - Add a default label to all pods.
    - Attach pod security policy for a pod that is created without any pod security policy configuration.
- **Generate:** Create or update Kubernetes resources dynamically.. For example;
    - Create a default network policy whenever a namespace is created.
- **Verify Images:** Validate container image signatures. For example;
    - Verify if the Images used in the pod resources are properly signed and verified images.

## How Kyverno Works

1. **User Action:** 
    - A developer tries to create or update a Kubernetes resource (e.g., Pod).
2. **API Server Interaction:**
    - The request is sent to the Kubernetes API server.
3. **Validation by Kyverno:**
    - Kyverno compares the request with defined policies.
    - If the resource violates policies, Kyverno can
        - Block the request (Enforce).
        - Audit the violation (Audit).
4. **Approval or Rejection:**
    - Based on the policy evaluation, the resource is created or rejected.

![image](https://user-images.githubusercontent.com/43399466/219931795-dce93e3b-9f78-42ef-ba5e-9aa685252e2f.png)

## Kyverno Policy Structure

A Kyverno Policy is a collection of rules, and each rule contains:
- **Match Declaration:** Specifies the resources the rule applies to.
- **Exclude Declaration** (Optional): Excludes specific resources from the rule.
- **Action Declaration:** Each rule must contain one of the following:
    - Validate: Ensure resources conform to defined standards.
    - Mutate: Modify resource definitions automatically.
    - Generate: Create or update additional resources.
    - VerifyImages: Validate image signatures and sources.

**Rule Limitation:**
- Each rule can only contain one of the four action types (validate, mutate, generate, or verifyImages).

![image](https://user-images.githubusercontent.com/43399466/219931973-14c0f501-ae49-4cab-9da5-b01950cc308f.png)

## Policy Scope

- **ClusterPolicy:** Applies cluster-wide across all namespaces.
- **Policy:** Applies only within a specific namespace.

**Key Difference:**
- The only distinction between ClusterPolicy and Policy is the scope of application.

**Additional Policy Types**
- PolicyException: Used to bypass or exclude specific resources from a policy.
- ClusterCleanupPolicy: Manages cleanup tasks across cluster-wide resources.

