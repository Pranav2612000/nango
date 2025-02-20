import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Core concepts

If you are new to Nango Sync please read this briefly: It will introduce all of the important concepts that help you get the most out of Sync fast.

[![Nango Sync Core concepts](/img/sync-core-concepts.png)](/img/sync-core-concepts.png)

## Syncs: How Nango moves your data
**Syncs** are the main objects in Nango: They represent a **continuous task** that periodically pulls data from an external API and syncs it to your database.

Syncs contain two important parts:
- Where and how to **fetch the data** from the external API
- Where and how to **transform and store the data** once it has been fetched

:::tip
Each Sync periodically performs HTTP requests to a specific external endpoint, for a specific account. The account's auth token is inserted in HTTP requests by Nango.
:::

The fastest way to understand Syncs is with some quick facts:
- You [create Syncs](manage-syncs.md) (and manage them) from your application's code using the Nango SDK or REST API
- You can have as many Syncs as you want
- Syncs periodically write the data they fetch to a database & table of your choosing (more on this below)
- Syncs automatically deal with paginated data, OAuth token refreshes etc.

Learn more about [creating & managing Syncs](sync-all-options.md) or look at the [full example of a Sync](#exampleSync) below.

## Storing & accessing the synced data

Accessing the data synced by Nango is easy: Just read it from your database.

Every record contains a timestamp of when it was last updated, so fetching changes is also easy. Nango can tell your application every time a sync refresh (called a "sync job" in Nango) has finished and how many records have been inserted/updated.

Nango also handles the data transformation from JSON to SQL (in addition to storing the raw data):
- Nango automatically transforms the synced data from [JSON to SQL](schema-mappings.md)
- You can [attach Metadata](sync-metadata.md) to every record Nango syncs in (e.g. user id, account id)
- You decide the destination table(s), for each Sync, Nango syncs the data to


This means that you can **tell Nango to write the data from many Syncs to the same database table**, so your application only has a single table to query (e.g. fetch HubSpot contacts, GitHub repos, Google Calendar events etc). And thanks to the attached metadata it is easy to know which records belong to which user, company or anything else that matters to your application.

The easiest way to see this all is with a simple example.

## Full example of a Nango Sync {#exampleSync}

Let's assume we have a SaaS application where users can signup and import all the Stargazers of their GitHub repos, so we can let them filter them, show them in the UI etc.

Because the [GitHub API endpoint](https://docs.github.com/en/rest/activity/starring#list-stargazers) to fetch stargazers is per repo we will setup one Sync in Nango per user per GitHub repo.

In practice it looks like this:
<Tabs groupId="programming-language">
  <TabItem value="node" label="Node SDK">

```ts
async function addStargazersSync(owner, repo, user_id) {
    
    let config = {
        headers: {                                    // HTTP headers to be sent with every API request
            'Accept': 'application/vnd.github+json'   
        },
        paging_header_link_rel: 'next'                // For pagination
        unique_key: 'id',                             // For deduping records
        mapped_table: 'github_stargazers',            // Name of the destination table
        metadata: {                                   // Metadata that will get attached to every synced row
            user_id: user_id,                         
            github_org: owner,                        
            github_repo: repo                         
        },
    };

    new Nango().sync(`https://api.github.com/repos/${owner}/${repo}/stargazers`, nango_options); 
}

// Add a Sync each for the nango repo and the nango-sync repo for the user with id 1
addStargazersSync('NangoHQ', 'nango', 1);
addStargazersSync('NangoHQ', 'nango-sync', 1);
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
    "headers": {
      "Accept": "application/vnd.github+json"
    },
    "paging_header_link_rel": "next",
    "unique_key": "id",
    "mapped_table": "github_stargazers",
    "metadata": {
      "user_id": 1,
      "github_org": "NangoHQ",
      "github_repo": "nango"
    }
  }'
# headers: HTTP headers to be sent with every API request
# paging_header_link_rel: For pagination
# unique_key: For deduping records
# mapped_tabl: Name of the destination table
# metadata: Metadata that will get attached to every synced row
  ```

</TabItem>
</Tabs>

In our database Nango creates a single table called `github_stargazers`, which contains the data from both Syncs (much simplified here):
```plaintext
github_stargazers
┌────────────────────────────┬─────────┬─────────────┬─────────┬──────────────────────────────────┐
│         emitted_at         │ user_id │ github_repo │  login  │            avatar_url            │
├────────────────────────────┼─────────┼─────────────┼─────────┼──────────────────────────────────┤
│ 2022-12-07 14:01:52.019+00 │       1 │ nango       │ sradu   │ https://avatars.githubusercon... │
│ 2022-12-07 14:01:52.028+00 │       1 │ nango       │ bastien │ https://avatars.githubusercon... │
│ 2022-12-07 14:01:52.093+00 │       1 │ nango-sync  │ sradu   │ https://avatars.githubusercon... │
└────────────────────────────┴─────────┴─────────────┴─────────┴──────────────────────────────────┘
```
Remember that Syncs are continuous, so Nango will automatically keep the data in this table up to date even as more people star the repos (or stars get removed).


In our application we can now run any SQL query we want to fetch the data and use it in our application:
```sql
-- Fetching all stargazers from all repos of user with id 1
SELECT * FROM github_stargazers WHERE user_id = 1;

-- Count the stargazers per repo for all repos of user with id 1
SELECT github_org, github_repo, COUNT(*)
FROM github_stargazers
WHERE user_id = '1'
GROUP BY github_org, github_repo;
```