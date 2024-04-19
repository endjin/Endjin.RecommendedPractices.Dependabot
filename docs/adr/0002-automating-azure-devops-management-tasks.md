# Automating Azure DevOps Management Tasks via CodeOps

## Status

Proposed

## Context

Using a [GitOps][1] approach for managing routine configuration updates for Azure DevOps offers the possibility of:
* Devolving responsibility for such changes to those with the requirements
* Scaling-out the management overhead of making such changes
* Maintaining the required oversight via the pull request mechanism
* Strong audit trail of configuration changes

Here are some example scenarios:
1. Managing access to projects
1. Managing service connections and their security requirements

The premise of these [CodeOps][2] solutions relies on using a trusted Azure Active Directory (AAD) service principal to apply configuration changes instead of users running scripts from their computer or using UI tools.

* Users have access to the git repository and can request changes via pull request
* Administrators have approval oversight of those pull requests
* Once approved & merged an automated process runs to apply the updated configuration state from the `main` branch
* Admin access to the automated process itself is tightly controlled and the credential for the trusted identity should never be known outside of the initial registration process of the automation platform (e.g. registering the service connection with Azure DevOps)
* The repository history provides an audit trail of configuration changes

This approach has already been used in several scenarios, but never where the configuration being managed related to an Azure DevOps instance.

Currently it is not possible to grant an AAD service principal access to Azure DevOps - [documentation-reference][2].

This blocks the current approach, that would otherwise use an Azure DevOps pipeline to execute the CodeOps process.

The remainder of this ADR outlines two approaches for resolving this issue:
* Creating a 'service account' user in AAD that the automated process uses to authenticate to Azure DevOps
* Use AAD delegation to allow an AAD application to impersonate a calling user

### AAD Delegation
This fundamentally changes the approach and would necessitate a move towards the CodeOps process running as some kind of application that can be interacted with by other parties that have an AAD user context.

This would seem to clash with the goal of not requiring users to have the permissions needed to run the process.


### Service Account
Whilst it would be trivial to setup a dedicated AAD user account with the required permissions to run the process, such an identity cannot always be used to natively execute a pipeline within CI/CD tools (e.g. Azure DevOps, GitHub Actions).

Therefore it would be necessary for the pipeline to perform a 'runas' operation when communicating with the Azure DevOps REST API.  This in turn means that the credential for this user needs to be retrievable (e.g. from key vault), but this clashes with the goal to minimise the surface area for attacking the trusted identity's credential.

This could be mitigated by treating the password for the AAD user account as 'disposable', by allowing the automated process to generate a new password before use.  For example:
* As before, the pipeline runs as a service principal (SP)
* The SP now requires additional AAD permissions, making it an 'owner' of the service account AAD user
* During the run, the pipeline enables the user and resets the password to a randomly-generated value and is only retained in-memory
* The pipeline then uses those details to authenticate to the Azure DevOps REST API
* Once completed, the SP could optionally disable the service account AAD user and/or reset the password once again


## Decision

TBC

## Consequences

### Positive
* It is possible to perform automated processes against the Azure DevOps REST API (either directly or via other tooling e.g. azure-cli)
* No credentials need to be persisted outside of the platform running the pipeline - this includes the service principal used by the pipeline as well as the AAD service account

### Negative
* Affected CodeOps processes become more complicated by needing to handle the alternative identity for the parts of their process that relate to Azure DevOps
* The AAD 'service account' user will require an Azure DevOps license - where multiple such accounts are used (e.g. to apply separation of responsibility) each one will require its own license
* The AAD service principal requires additional AAD permissions
* AAD users with permissions to manage user accounts could gain access to the service account and the permissions it has been granted
    * NOTE: There is an existing equivalent attack vector for the service principal, by users with access to manage AAD applications


[1]: https://www.gitops.tech
[2]: https://endjin.com/blog/2020/11/does-your-github-repo-need-code-operations
[3]: https://docs.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/authentication-guidance?view=azure-devops