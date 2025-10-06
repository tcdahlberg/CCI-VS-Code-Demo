# Snapshot Scratch Orgs

Salesforce CLI provides a way to create, manage, and utilize snapshot scratch orgs. Snapshots allow you to quickly spin up scratch orgs with predefined configurations, making it easier to test and develop in consistent environments.

**Creation**
```bash
sf org create snapshot --name alias_for_your_snapshot --source-org Scratch_org_name_created_to_snapshot --target-dev-hub dev_org_alias --description "Description of what the snapshot contains"
```

**Deletion**
```bash
sf org delete snapshot --name name_of_snapshot --target-dev-hub dev_org_alias
```

**Listing**
```bash
sf org list snapshots --target-dev-hub dev_org_alias
```

**Usage**
Simply use your snapshot alias in your `project-scratch-def.json` file:

```bash
{
  "orgName": "Salesforce",
  "snapshot": "alias_for_your_snapshot"
}
```