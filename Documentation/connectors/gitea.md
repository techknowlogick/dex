# Authentication through Gitea

## Overview

One of the login options for dex uses the Gitea OAuth2 flow to identify the end user through their Gitea account.

When a client redeems a refresh token through dex, dex will re-query Gitea to update user information in the ID Token. To do this, __dex stores a readonly Gitea access token in its backing datastore.__ Users that reject dex's access through Gitea will also revoke all dex clients which authenticated them through Gitea.

## Configuration

Register a new OAuth consumer with [Gitea](https://docs.gitea.io/en-us/oauth2-provider/) ensuring the callback URL is `(dex issuer)/callback`. For example if dex is listening at the non-root path `https://auth.example.com/dex` the callback would be `https://auth.example.com/dex/callback`.

The following is an example of a configuration for `examples/config-dev.yaml`:

```yaml
connectors:
- type: gitea
  # Required field for connector id.
  id: gitea
  # Required field for connector name.
  name: Gitea
  config:
    # Credentials can be string literals or pulled from the environment.
    clientID: $GITEA_CLIENT_ID
    clientSecret: $GITEA_CLIENT_SECRET
    redirectURI: http://127.0.0.1:5556/dex/callback
    # optional, default = https://gitea.com
    baseURL: https://gitea.com

    # Optional organizations and teams, communicated through the "groups" scope.
    #
    # NOTE: This is an EXPERIMENTAL config option and will likely change.
    #
    # Dex queries the following organizations for group information if the
    # "groups" scope is provided. Group claims are formatted as "(org):(team)".
    # For example if a user is part of the "engineering" team of the "coreos"
    # org, the group claim would include "coreos:engineering".
    #
    # If orgs are specified in the config then user MUST be a member of at least one of the specified orgs to
    # authenticate with dex.
    #
    # If 'orgs' is not specified in the config and 'loadAllGroups' setting set to true then user
    # authenticate with ALL user's Gitea groups. Typical use case for this setup:
    # provide read-only access to everyone and give full permissions if user has 'my-organization:admins-team' group claim.
    orgs:
    - name: my-organization
    - name: my-organization-with-teams
      # A white list of teams. Only include group claims for these teams.
      teams:
      - red-team
      - blue-team
```

