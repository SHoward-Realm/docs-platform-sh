# Operational Errors

Occasionally it can be useful for debugging purposes to check the Realm Object Server logs.  Realm Object Server produces two specific classes of error and warning diagnostics that may be useful to system admins and developers.

| **Error Message** | **Solution** |
| :--- | :--- |
| Failed to accept a connection due to the file descriptor limit, consider increasing the limit in your system config | Increase the file descriptor limit on your machine.  Instructions on how to do this can be found [here](https://www.tecmint.com/increase-set-open-file-limits-in-linux/).  In general, you'll want to set this well above the number of realms on your server \(3-5x the number of realms is typically a good place to start\). |
| mmap\(\) failed: Cannot allocate memory  | Consider increasing the amount of memory available on your server.  If you believe that your server has ample resources, please [contact us](https://support.realm.io/).   |

