
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Examples of Nango Sync

:::info Nango works with any API that returns JSON
The examples on this page are just a handful of the millions of APIs Nango supports out of the box, it is far from a complete list.

If you are unsure if your API is supported by Nango feel free to try it (we found that by now 80%+ of APIs work out of the box) or ask us on the [Slack community](https://nango.dev/slack): We are happy to help!
:::

A collection of cool things people have built with Nango.  
The sidebar to the right has an index so you can directly jump to your favorite 👉

* Want to run one of these? You can find these (and more) all ready to run in the [`/examples`](https://github.com/NangoHQ/nango-sync/tree/main/examples) in the repo.
* Need help with one of these (or another API/endpoint)? Reach out on our [Community Slack](https://nango.dev/slack), we are online all day and happy to help!
* Want to share yours? [Edit this file and submit a PR!](https://github.com/nangohq/nango-sync/tree/main/docs/docs/real-world-examples.md)

## Reddit: Sync all posts/submissions from a subreddit

**Endpoint docs:**  
https://www.reddit.com/dev/api/#GET_new

**Nango Sync config to sync all submissions from a subreddit to your local database:**
<Tabs groupId="programming-language">
  <TabItem value="node" label="Node SDK">

```ts
import { Nango } from '@nangohq/node-client'

let config = {
        friendly_name: 'Reddit Subreddit Posts',                   // Give this Sync a name for prettier logs.
        mapped_table: 'reddit_posts',                              // Name of the destination SQL table
        response_path: 'data.children',                            // For finding records in the API response.
        paging_cursor_object_response_path: 'data.name',           // For finding pagination data in responses.
        paging_cursor_request_path: 'after',                       // For adding pagination data in requests.
        max_total: 100,                                            // For fetching limited records while testing.
        frequency: '1 minute'                                      // How often sync jobs run in natural language.
    };

new Nango().sync('https://www.reddit.com/r/glastonbury_festival/new.json', config);  // Replace 'glastonbury_festival' with your subreddit
```
  </TabItem>
  <TabItem value="curl" label="REST API (curl)">

  ```bash
  curl --request POST \
--url http://localhost:3003/v1/syncs \
 --header "Content-type: application/json" \
 --data '
 {
"url": "https://www.reddit.com/r/glastonbury_festival/new.json",
"friendly_name": "Reddit Subreddit Posts",
"mapped_table": "reddit_posts",
"response_path": "data.children",
"paging_cursor_object_response_path": "data.name",
"paging_cursor_request_path": "after",
"max_total": 100,
"frequency": "1 minute"
}'

# url: Replace 'glastonbury_festival' with your subreddit.
# friendly_name: Give this Sync a name for prettier logs.
# mapped_table: Name of the destination SQL table
# response_path: For finding records in the API response.
# paging_cursor_object_response_path: For finding pagination data in responses.
# paging_cursor_request_path: For adding pagination data in requests.
# max_total: For fetching limited records while testing.
# frequency: How often sync jobs run in natural language.
  ```
  </TabItem>
</Tabs>

**Run the example ▶️**  
You can run this example from the `nango` folder root with:
```bash
npm run example syncRedditSubredditPosts <subreddit>
```

## Slack: Sync all posts from a Slack channel

**Endpoint docs:**  
https://api.slack.com/methods/conversations.history

**Nango Sync config to sync all posts from a Slack channel to your local database:**
<Tabs groupId="programming-language">
  <TabItem value="node" label="Node SDK">

```ts
import { Nango } from '@nangohq/node-client'

let app_token = "fake-token";        // Replace with your Slack app token.
let channel_id = 'fake-id';          // Replace with the ID of the channel you want to sync.

let config = {
        friendly_name: 'Slack Messages',                                        // Give this Sync a name for prettier logs.
        mapped_table: 'slack_messages',                                         // Name of the destination SQL table
        response_path: 'messages',                                              // For finding records in the API response.
        paging_cursor_metadata_response_path: 'response_metadata.next_cursor',  // For finding pagination data in responses.
        paging_cursor_request_path: 'cursor',                                   // For adding pagination data in requests.          
        headers: { authorization: `Bearer ${app_token}` },                      // Replace with your Slack app token
        query_params: { channel: channel_id },                                  // Replace with the ID of the channel
        max_total: 100,                                                         // For fetching limited records while testing.
        frequency: '1 minute'                                                   // How often sync jobs run in natural language.
    };

new Nango().sync('https://slack.com/api/conversations.history', config); 
```
  </TabItem>
  <TabItem value="curl" label="REST API (curl)">

  ```bash
  curl --request POST \
--url http://localhost:3003/v1/syncs \
 --header "Content-type: application/json" \
 --data '
{
"url": "https://slack.com/api/conversations.history",
"friendly_name": "Slack Messages",
"mapped_table": "slack_messages",
"response_path": "messages",
"paging_cursor_metadata_response_path": "response_metadata.next_cursor",
"paging_cursor_request_path": "cursor",
"headers": { "authorization": "Bearer [APP-TOKEN]" },
"query_params": { "channel": "[CHANNEL-ID]" },
"max_total": 100,
"frequency": "1 minute"
}'

# url: external API endpoint to use.
# friendly_name: Give this Sync a name for prettier logs.
# mapped_table: Name of the destination SQL table
# response_path: For finding records in the API response.
# paging_cursor_metadata_response_path: For finding pagination data in responses.
# paging_cursor_request_path: For adding pagination data in # requests.          
# headers: Replace with your Slack app token
# query_params: Replace with the ID of the channel
# max_total: For fetching limited records while testing.
# frequency: How often sync jobs run in natural language.
  ```
  </TabItem>
</Tabs>

**Run the example ▶️**  
You can run this example from the `nango` folder root with:
```bash
npm run example syncSlackMessages <oauth_token> <channel_id>
```

## Github: Sync all stargazers from a repo

**Endpoint docs:**  
https://docs.github.com/en/rest/activity/starring#list-stargazers

This example syncs the stargazers of multiple different repos (and users) into a single table (we use `github_stargazers` here). It also adds metadata attributes, which get attached to every synced record.

**Nango Sync config to sync all stargazers from a repo to your local database:**
<Tabs groupId="programming-language">
  <TabItem value="node" label="Node SDK">

```ts
import { Nango } from '@nangohq/node-client'

let owner = 'nangohq';  // Replace with your github account
let repo = 'nango';     // Replace with your repo

let config = {
    friendly_name: 'Github Stargazers',                // Give this Sync a name for prettier logs.
    mapped_table: 'github_stargazers',                 // Name of the destination SQL table.
    metadata: {                                        // Metadata that will get attached to every synced row.
        github_org: owner,                             // The GitHub org.
        github_repo: repo                              // The repo name.
    },
    unique_key: 'id',                                  // The key of the unique id in the records, for upserts.
    headers: {                                         // HTTP headers to be sent with every API request.
        'Accept': 'application/vnd.github+json'        // GitHub recommends passing this for every API request.
    },
    paging_header_link_rel: 'next',                    // For pagination.
    max_total: 100,                                    // For fetching limited records while testing.
    frequency: '1 minute'                              // How often sync jobs run in natural language.
};

new Nango().sync('https://api.github.com/repos/${owner}/${repo}/stargazers', config); 
```
  </TabItem>
  <TabItem value="curl" label="REST API (curl)">

  ```bash
  curl --request POST \
--url http://localhost:3003/v1/syncs \
 --header "Content-type: application/json" \
 --data '
 {
"url": "https://api.github.com/repos/nangohq/nango/stargazers",
"friendly_name": "Github Stargazers",
"mapped_table": "github_stargazers",
"metadata": { "github_org": "NangoHQ", "github_repo": "nango" },
"unique_key": "id",
"headers": { "Accept": "application/vnd.github+json" },
"paging_header_link_rel": "next",
"max_total": 100,
"frequency": "1 minute"
}'

# url: external API endpoint to use.
# friendly_name: Give this Sync a name for prettier logs.
# mapped_table: Name of the destination SQL table.
# metadata: Metadata that will get attached to every synced row.
# unique_key: The key of the unique id in the records, for upserts.
# headers: HTTP headers to be sent with every API request.
# paging_header_link_rel: For pagination.
# max_total: For fetching limited records while testing.
# frequency: How often sync jobs run in natural language.
  ```
  </TabItem>
</Tabs>

**Run the example ▶️**  
You can run this example from the `nango` folder root with:
```bash
npm run example syncGithubStargazers <owner> <repo>
```


## HubSpot: Sync all HubSpot (CRM) contacts

**Endpoint docs:**  
https://developers.hubspot.com/docs/api/crm/contacts  
(click on the "Endpoints" tab, the use the dropdown to find the endpoint or scroll down)

**Nango Sync config to sync contacts from the HubSpot CRM to your local database:**
<Tabs groupId="programming-language">
  <TabItem value="node" label="Node SDK">

```ts
import { Nango } from '@nangohq/node-client'

let api_token = "fake-token";      // Your Hubspot API token

let config = {
    friendly_name: 'Hubspot Contacts',                         // Give this Sync a name for prettier logs.
    mapped_table: 'hubspot_contacts',                          // Name of the destination SQL table
    method: NangoHttpMethod.GET,                               // Required info to query the right endpoint.
    headers: { authorization: `Bearer ${api_token}` },         // For auth.
    query_params: { limit: 100 },                              // Get 100 records per page (HubSpot API setting)
    paging_cursor_request_path: 'after',                       // For adding pagination data in requests.
    paging_cursor_metadata_response_path: 'paging.next.after', // For finding pagination data in responses.
    response_path: 'results',                                  // For finding records in the API response.
    unique_key: 'id',                                          // Provide response field path for deduping records.
    max_total: 100,                                            // For fetching limited records while testing.
    frequency: '1 minute'                                      // How often sync jobs run in natural language.
};

new Nango().sync('https://api.hubapi.com/crm/v3/objects/contacts', config);
```
  </TabItem>
  <TabItem value="curl" label="REST API (curl)">

  ```bash
  curl --request POST \
--url http://localhost:3003/v1/syncs \
 --header "Content-type: application/json" \
 --data '
 {
"url": "https://api.hubapi.com/crm/v3/objects/contacts/search",
"friendly_name": "Hubspot Contacts",
"method": "POST",
"headers": { "Authorization": "Bearer [YOUR-ACCESS-TOKEN]"},
"paging_cursor_request_path": "after",
"paging_cursor_metadata_response_path": "paging.next.after",
"response_path": "results",
"unique_key": "id",
"max_total": 100,
"frequency": "1 minute"
}'

# friendly_name: 'Give this Sync a name for prettier logs.
# mapped_table: Name of the destination SQL table
# method: Required info to query the right endpoint.
# headers: Add the relevant auth header.
# paging_cursor_request_path: For adding pagination data in requests.
# paging_cursor_metadata_response_path: For finding pagination data in responses.
# response_path: For finding records in the API response.
# unique_key: Provide response field path for deduping records.
# max_total: For fetching limited records while testing.
# frequency: How often sync jobs run in natural language.
  ```
  </TabItem>
</Tabs>

**Run the example ▶️**  
You can run this example from the `nango` folder root with:
```bash
npm run example syncHubspotContacts <oauth_token>
```
