# Retrieve Target URL from Platform.sh for PR Environment
Polls the [GitHub Status API endpoint](https://docs.github.com/en/enterprise-server@3.0/rest/reference/commits#commit-statuses) 
for a Pull Request external integration until `success` is posted, and returns the value for `target_url`. 

## Inputs
* `repository` - Owner/namespace of the target repository. Defaults to `github.repository_owner` from the 
[github context](https://docs.github.com/en/actions/learn-github-actions/contexts#github-context).  
* `github-token` - Github [personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) with access rights to the target repository so we can work with the github api. **REQUIRED**.
* `ref` - The branch or tag ref that we should check for a status. Defaults to `github.head_ref` from the 
[github context](https://docs.github.com/en/actions/learn-github-actions/contexts#github-context). 
* `timeout` - How long (in seconds) should we continue checking for the new environment? Must be whole (integer) 
seconds. Defaults to 300.
* `delay-in-seconds` - The delay in seconds between each check. Must be whole (integer) seconds. Defaults to 10. 

## Outputs
* `target_url` - Target URL as returned by the external (Platform.sh) integration.

## Example usage:
See `Wait for psh and get target url` step below
```yaml
name: "smoke test"
on:
    workflow_run:
        workflows: ["platformsh"]
        types:
            - completed
    pull_request:
        branches:
            - main

jobs:
  test-pr-env:
    name: Test the PR environment
      runs-on: ubuntu-latest
      steps:
        - name: 'Wait for psh and get target url'
          id: get-target-url
          uses: gilzow/retrieve-psh-prenv-url@main
          with:
            github-token: ${{ secrets.GH_TOKEN }}
        - name: 'Does Our App Work?'
          id: smokescreen
          run: |
            target_url=${{ steps.get-target-url.outputs.target_url }}
            #we dont need the whole response (yet), just see if we get a 200
            response=$(curl -s -o /dev/null -I -w "%{http_code}" "${target_url//http:/https:}");
            echo "::notice::Response from server for our PR environment is ${response}"
```
