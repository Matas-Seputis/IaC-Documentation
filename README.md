# IaC (Infrastructure as Code) Repository for the GCP Landing Zone - GitHub
GitHub is a web-based platform that provides version control using Git. This repository is designed to automate the provisioning and management of GCP infrastructure components necessary for a landing zone. A landing zone is a secure, scalable, and robust environment that serves as a foundation for cloud operations. By using this IaC repository, we ensure that the GCP landing zone is set up consistently across different environments. Here are some key features provided by GitHub:
<lu>
<li><strong>Version Control</strong>: GitHub uses Git, a distributed version control system, to track changes in source code during software development. This allows multiple contributors to work on projects simultaneously without conflicts. </li>
<li><strong>Collaboration</strong>: GitHub facilitates collaboration among developers through features like pull requests, code reviews, and issue tracking. Teams can work together on code, discuss changes, and resolve issues efficiently.</li>
<li><strong>Code Hosting</strong>: GitHub hosts repositories, providing a centralized place to store code and documentation. This ensures that code is easily accessible and securely stored.</li>
<li><strong>CI/CD Integration</strong>: GitHub integrates with Continuous Integration and Continuous Deployment (CI/CD) tools, allowing for automated testing, building, and deployment of code.</li>
<li><strong>Project Management</strong>: GitHub offers project management tools such as GitHub Projects, which help in planning, tracking, and managing work across teams.
Security: GitHub provides security features such as Dependabot for automated dependency updates, secret scanning, and security advisories to protect your codebase.</li>

# Authentication to Google Cloud
When you access Google Cloud services by using the Google Cloud CLI, Cloud Client Libraries, tools that support Application Default Credentials (ADC) like Terraform, or REST requests, use the following diagram:

![GCP Authentication Method diagram](https://github.com/Matas-Seputis/terraform-automation/assets/72572593/f88c9854-d6b0-4503-876f-2a1fe77f36f7)

Since our code repository is GitHub (external repo) and within the organization development environment - Workload Identity Federation vs. Service Account Keys.

Traditionally, authenticating from GitHub to Google Cloud required exporting and storing a long-lived JSON service account key, turning an identity management problem into a secrets management problem. Not only did this introduce additional security risks if the service account key were to leak, but it also meant developers would be unable to authenticate from GitHub to Google Cloud if their organization has disabled service account key creation (a common security best practice).
Workload Identity Federation eliminates the maintenance and security burden associated with service account keys. 
So we will go with <strong>Workload Identity Federation</strong>

## Workload identity pool providers
A workload identity pool provider is an entity that describes a relationship between Google Cloud and your IdP, including the following:
<lu>
<li>AWS</li>
<li>Azure AD</li>
<li>GitHub</li>
<li>GitLab</li>
<li>Kubernetes clusters</li>
<li>Okta</li>
<li>On-premises Active Directory Federation Services (AD FS)</li>
<li>Terraform</li>
</lu>

Workload Identity Federation follows the OAuth 2.0 token exchange specification. You provide a credential from your IdP to the Security Token Service, which verifies the identity on the credential, and then returns a federated token in exchange.

## GitHub to GCP via OpenID (OIDC)
OpenID Connect (OIDC) allows your GitHub Actions workflows to access resources in Google Cloud Platform (GCP), without needing to store the GCP credentials as long-lived GitHub secrets.

This guide gives an overview of how to configure GCP to trust GitHub's OIDC as a federated identity, and includes a workflow example for the google-github-actions/auth action that uses tokens to authenticate to GCP and access resources.
Link: https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-google-cloud-platform

# Service Accounts
Service accounts are special types of Google accounts intended to represent non-human users that need to authenticate and be authorized to access Google Cloud resources. They are used to provide credentials for automated processes, such as running applications or executing scripts that need to interact with Google Cloud services.

In a CI/CD pipeline, specifically when using GitHub Actions to manage Terraform workflows service accounts play a crucial role:
<strong>Authentication and Authorization:</strong>
<lu>
<li><strong>Authentication</strong>: Service accounts provide a secure and automated way for GitHub Actions to authenticate to GCP. They eliminate the need for storing and managing user credentials.</li>
<li><strong>Authorization</strong>: Service accounts can be assigned specific roles and permissions, ensuring that the CI/CD pipeline only has access to the necessary resources and operations.</li>
</lu>

<strong>Security:</strong>
<lu>
<li>Service accounts reduce the risk of exposing user credentials in the CI/CD environment. By binding service accounts to Workload Identity Federation pool, the CI/CD workflow can authenticate securely to GCP.
Roles and permissions associated with service accounts can be tightly controlled, following the principle of least privilege, which minimizes the potential impact of a compromised account.</li>
</lu>

<strong>Automation:</strong>
<lu>
<li>CI/CD pipelines require non-interactive authentication to perform tasks like tf-plan (generating and showing the execution plan for Terraform) and tf-apply (applying the changes required to reach the desired state of the configuration). Service accounts enable this automation by providing a way for the pipeline to authenticate programmatically.</li>
</lu>

<strong>Consistency and Reliability:</strong>
<lu>
<li>Using service accounts ensures consistent and reliable authentication across different stages of the CI/CD pipeline. This consistency is crucial for maintaining the integrity and reliability of automated deployments and infrastructure management.</li>
</lu>

# ViaPlay Workflow Explanation:

![image](https://github.com/Matas-Seputis/terraform-automation/assets/72572593/4fe2f242-9296-45ef-a944-21d8d849b6b2)

This diagram illustrates how GitHub Actions can securely interact with Google Cloud resources using Workload Identity Federation.

<strong>GitHub Actions Workflow and OIDC Provider Initiation:</strong>
<lu>
<li><strong>GitHub Actions Workflow</strong>: A GitHub Actions workflow is triggered by an event (e.g., a pull request or a push to a branch).</li>
<li><strong>OIDC Token Request</strong>: The GitHub Actions workflow requests an OpenID Connect (OIDC) token from the GitHub OIDC provider. This token includes metadata about the workflow (e.g., repository name, run ID, event name).</li>

<strong>Workload Identity Pool and Security Token Services (STS):</strong>
<lu>
<li><strong>Workload Identity Pool</strong>: Google Cloud uses a Workload Identity Pool to establish trust between the external identity (GitHub) and Google Cloud.</li>
<li><strong>Security Token Services (STS)</strong>: The OIDC token from GitHub is sent to Google Cloud’s Security Token Services (STS). STS validates the token against the Workload Identity Pool, ensuring that it matches the configured trust and access policies.</li>
</lu>

<strong>Token Exchange and Service Account Impersonation:</strong>
<lu>
<li><strong>Token Exchange</strong>: Once STS validates the OIDC token, it exchanges this token for a short-lived access token. This access token is then used to impersonate a Google Cloud service account.</li>
<li><strong>Service Account</strong>: The service account is configured with specific roles and permissions to access Google Cloud resources. This setup ensures that the GitHub Actions workflow has the necessary permissions without requiring long-lived service account keys.</li>
</lu>

<strong>Accessing Google Cloud Resources:</strong>
<lu>
<li><strong>Google Cloud Resources</strong>: Using the short-lived access token, the GitHub Actions workflow can interact with various Google Cloud resources (e.g., Compute Engine, Cloud Storage, BigQuery) as permitted by the service account’s roles and permissions.</li>
</lu>

## Benefits of such Workflow:
<strong>Security</strong>: Eliminates the need for storing and managing long-lived service account keys in GitHub. Uses short-lived tokens instead.
<strong>Automation</strong>: Allows seamless and secure authentication from GitHub Actions to Google Cloud.
<strong>Scalability</strong>: Supports managing access at scale, enabling consistent access policies across multiple workflows and repositories.
<strong>Compliance</strong>: Provides auditable trails of access and actions performed by the GitHub Actions workflow on Google Cloud resources.
