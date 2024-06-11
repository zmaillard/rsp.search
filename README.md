## Upgrade Guide
- Branch repo
- Fork the Volume `fly volumes fork <volume ID> --region <region>`.  Note the new volume id.
- Clone existing machine and attach new volume:
```
fly machine clone <machine id> --region <region code> --attach-volume <volume id>:<destination mount path>
```
- Branch repo
- Update `fly.toml` to point to the new Meilisearch Docker Image.  Update the application name to be something unique.  For instance:
```
app = 'rsp-search-v1.8.2'
image = 'getmeili/meilisearch:v1.8.2'
```


1. Create Snapshot of the volume mounted on the current machine: `fly volumes snapshots create <volume_id>`.  Find volume id from the fly dashboard, or from the console (show command).  Wait for snapshot to be created. Note the new Id.
2. Create new branch for the upgrade.
3. Update the `fly.toml` file to point to the new Meilisearch Docker Image.  Update the application name to be something unique.  Comment out the `[[mounts]]` section initially.
4. Deploy new application with `fly launch`.  Confirm defaults.
5. Create new volume from snapshot on step 1. `fly volumes create --region sea -a <new_app_name> --snapshot-id <snapshot_id_from_step_1>`


- Create dump of the database (https://search.roadsign.pictures/dumps POST)
- Check status of the Dump - https://search.roadsign.pictures/tasks/63803 GET .  It will be ready when status is "succeeded".  Take note of the dump file that was generated under details -> dumpUid.
- SSH into machine `fly ssh console` and navigate to the `dumps` directory.  Verify the dump file created above with the `dumpUid` value is in the `dumps` directory.
- Pull the dump file off the volume using sftp `fly ssh sftp get /meili_data/dumps/<dump_uid>.dump`

- Branch repo
- Update `fly.toml` to point to the new Meilisearch Docker Image.  Update the application name to be something unique.  For instance:
```
app = 'rsp-search-v1.8.2'
image = 'getmeili/meilisearch:v1.8.2'
```

- Launch new apps with `fly launch`.  Confirm that you wish to copy its configuration to the new app.
- Confirm a volume was created for the new app with `fly volumes list`.  Note the new volume id.
- Extend the volume size to 3GB `fly vol extend <vol_id> -s 3`
- Upload dump file:
```
fly sftp console
cd /meili_data/dumps
put <dump_file>
<ctrl+C>
```
- SSH into the new machine `fly ssh console`
```
rm -Rf /meili_data/data.ms
meilisearch --import-dump dumps/20240611-142338344.dump
```

- Note the new master key that was generated.  Create new secret in the Fly.IO dashboard for `MEILI_MASTER_KEY` with that value.  Note the new secret in your key manager.  Re-deploy the application.

- After deploy completes, visit new application using `fly.dev` domain and verify the data looks correct.

## Swap DNS
- Create a new certificate for the new application:
```
fly certs add search.roadsign.pictures
```
- Note the new CNAME and update the DNS record to point to the new application.
