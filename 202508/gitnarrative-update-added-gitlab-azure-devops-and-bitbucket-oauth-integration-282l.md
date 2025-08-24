---
Title: GitNarrative Update: Added GitLab, Azure DevOps, and Bitbucket OAuth Integration
Description: 
Author: Chris Moore
Date: 2025-08-24T18:03:40.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwelwmbpjqcmszwh01q4o.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwelwmbpjqcmszwh01q4o.png" alt=" " width="800" height="379"></a>Two weeks after launching GitNarrative with GitHub-only support, user feedback was clear: developers use multiple Git platforms. So I implemented OAuth for GitLab, Azure DevOps, and Bitbucket.</p>

<h2>
  
  
  The Implementation Challenge
</h2>

<p>Each platform required different approaches to OAuth and API integration:</p>

<h3>
  
  
  GitLab
</h3>

<ul>
<li>Similar OAuth flow to GitHub but different scoping</li>
<li>Commit metadata structure varies slightly</li>
<li>Self-hosted instances require dynamic endpoint configuration</li>
</ul>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
python
# GitLab OAuth configuration
GITLAB_OAUTH_CONFIG = {
    'client_id': settings.GITLAB_CLIENT_ID,
    'redirect_uri': f"{settings.BASE_URL}/auth/gitlab/callback",
    'scope': 'read_api read_repository',
    'response_type': 'code'
}
Azure DevOps

More complex authentication flow with organization context
Different API structure for repository and commit data
Required additional permissions for private repositories

python# Azure DevOps requires organization in the API calls
async def get_azure_repos(access_token: str, organization: str):
    headers = {'Authorization': f'Bearer {access_token}'}
    url = f'https://dev.azure.com/{organization}/_apis/git/repositories'
    # Additional complexity for handling organization context
Bitbucket

Unique repository structure compared to GitHub/GitLab
Different branch and pull request handling
Workspace-based organization model

API Adaptation Layer
The core git analysis engine remained unchanged, but I built an adapter layer:
pythonclass GitPlatformAdapter:
    @abstractmethod
    async def get_repositories(self, token: str) -&gt; List[Repository]:
        pass

    @abstractmethod 
    async def analyze_commits(self, repo_id: str, token: str) -&gt; CommitAnalysis:
        pass

class GitLabAdapter(GitPlatformAdapter):
    async def get_repositories(self, token: str):
        # GitLab-specific implementation

class AzureDevOpsAdapter(GitPlatformAdapter):
    async def get_repositories(self, token: str):
        # Azure-specific implementation
User Experience Considerations
Each platform has different user expectations:

GitLab users often work with self-hosted instances
Azure DevOps users expect tight Microsoft ecosystem integration
Bitbucket users are familiar with Atlassian workflow patterns

The OAuth flows needed to feel native to each platform while maintaining consistent GitNarrative branding.
Current Usage Distribution
After two weeks with multi-platform support:

GitHub: 82% (down from 100%)
GitLab: 12%
Azure DevOps: 4%
Bitbucket: 2%

GitLab adoption was faster than expected, likely due to its popularity in enterprise and open source communities.
Testing and Feedback Needed
Looking for developers using these platforms to test:

OAuth flow smoothness
Repository analysis accuracy
Story generation quality compared to GitHub

The git analysis algorithms work consistently across platforms, but each has unique metadata that could improve story generation.
Technical Lessons

OAuth complexity varies significantly between platforms
API rate limits differ - Azure DevOps is more restrictive
User expectations are platform-specific - don't assume GitHub patterns work everywhere
Error handling needs platform context - generic error messages confuse users

What's Next
Working on webhook integration for automatic story updates when repositories change. Each platform handles webhooks differently, so this will be another interesting integration challenge.

GitNarrative transforms git commit history into compelling project narratives. Try it with any of the supported platforms at gitnarrative.io
What Git platform do you use most, and what integration challenges have you faced building multi-platform tools?
</code></pre>

</div>

