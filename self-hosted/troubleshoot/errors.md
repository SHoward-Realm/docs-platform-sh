# Errors

## Session Specific Errors

| Error Message | Cause |
| :--- | :--- |
| 204 “Illegal Realm path \(BIND\)” | Indicates that the Realm path is not valid for the user. |
| 207 “Bad server file identifier \(IDENT\)” | Indicates that the local Realm specifies a link to a server-side Realm that does not exist. This is most likely because the server state has been completely reset. |
| 211 “Diverging histories \(IDENT\)” | Indicates that the local Realm specifies a server version that does not exists. This is most likely because the server state has been partially reset \(for example because a backup was restored\). |

## Client Level Errors

| Error Message | Cause |
| :--- | :--- |
| 105 “Wrong protocol version \(CLIENT\)” | The client and the server use different versions of the sync protocol due to a mismatch in upgrading. |
| 108 “Client file bound in other session \(IDENT\)” | Indicates that multiple sync sessions for the same client-side Realm file overlap in time. |
| 203 “Bad user authentication \(BIND, REFRESH\)” | Indicates that the server has produced a bad token, or that the SDK has done something wrong. |
| 206 “Permission denied \(BIND, REFRESH\)” | Indicates that the user does not have permission to access the Realm at the given path. |

## Operational Errors

Occasionally it can be useful for debugging purposes to check the Realm Object Server logs - which are output to the terminal by default but can easily be output to the file system or an external logger by extending your `index.ts`. Realm Object Server produces two specific classes of error and warning diagnostics that may be useful to system admins and developers.

| Error Message | Solution |
| :--- | :--- |
| Failed to accept a connection due to the file descriptor limit, consider increasing the limit in your system config | Increase the file descriptor limit on your machine.  Instructions on how to do this can be found [here](https://www.tecmint.com/increase-set-open-file-limits-in-linux/).   |
| mmap\(\) failed: Cannot allocate memory  | Consider increasing the amount of memory available on your server.  If you believe that your server has ample resources, please [contact us](https://support.realm.io/).   |







Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

