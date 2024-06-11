## Upgrade Guide
- Branch repo
- Create dump of the database (https://search.roadsign.pictures/dumps POST)
- Check status of the Dump - https://search.roadsign.pictures/tasks/63803 GET .  It will be ready when status is "succeeded".  Take note of the dump file that was generated under details -> dumpUid.
- SSH into machine `fly ssh console` and navigate to the `dumps` directory.  Verify the dump file created above with the `dumpUid` value is in the `dumps` directory.
- Fork the Volume `fly volumes fork <volume ID> --region <region>`.  Note the new volume id.
- Clone existing machine and attach new volume:
```
fly machine clone <machine id> --region <region code> --attach-volume <volume id>:<destination mount path>
```
- Update `fly.toml` to point to the new Meilisearch Docker Image.  Update the application name to be something unique.  For instance:
```
app = 'rsp-search-v1.8.2'
image = 'getmeili/meilisearch:v1.8.2'
```
