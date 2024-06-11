# Configuration for search.roadsign.pictures

## Upgrade Guide
1. Create dump of the database (https://search.roadsign.pictures/dumps POST)
2. Check status of the Dump - https://search.roadsign.pictures/tasks/63803 GET .  It will be ready when status is "succeeded".  Take note of the dump file that was generated under details -> dumpUid.
3. SSH into machine `fly ssh console` and navigate to the `dumps` directory.  Verify the dump file created above with the `dumpUid` value is in the `dumps` directory.
4. Pull the dump file off the volume using sftp `fly ssh sftp get /meili_data/dumps/<dump_uid>.dump`

5. Branch repo
```bash
git checkout main
git checkout -b upgrade/v1.8.2
```

6. Update `fly.toml` to point to the new Meilisearch Docker Image.  Update the application name to be something unique.  For instance:

```toml
app = 'rsp-search-v1.8.2'
image = 'getmeili/meilisearch:v1.8.2'
```

7. Launch new apps with `fly launch`.  Confirm that you wish to copy its configuration to the new app.
8. Confirm a volume was created for the new app with `fly volumes list`.  Note the new volume id.
9. Extend the volume size to 3GB `fly vol extend <vol_id> -s 3`
10. Upload dump file:
```bash
fly sftp console
cd /meili_data/dumps
put <dump_file>
<ctrl+C>
```
11. SSH into the new machine `fly ssh console`
```bash
rm -Rf /meili_data/data.ms
meilisearch --import-dump dumps/20240611-142338344.dump
```

- Note the new master key that was generated.  Create new secret in the Fly.IO dashboard for `MEILI_MASTER_KEY` with that value.  Note the new secret in your key manager.  Re-deploy the application.

- After deploy completes, visit new application using `fly.dev` domain and verify the data looks correct.

## Swap DNS
- Remove cert from old application
```
fly certs remove search.roadsign.pictures
```
- Create a new certificate for the new application:
```
fly certs add search.roadsign.pictures
```
- Note the new CNAME and update the DNS record to point to the new application.  Remove any old DNS records pointing to `search.roadsign.pictures`

## Cleanup
1. Save new Fly.IO configuration.
```bash
git commit -am "Upgrade to v1.8.2"
```

2. Remove old application.
```bash
git checkout main
fly destroy <old_app_name>
```

3. Merge new branch into main
```bash
git merge upgrade/v1.8.2
```
