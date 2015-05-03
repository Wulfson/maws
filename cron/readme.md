## Example cron definitions

#### eip_check
This will cause a check to see if your EIP is dangling, and associate it to the current instance if so.
```cron
10,40 * * * * root /home/ubuntu/maws/cron/eip_check <your eipalloc ID>
```

