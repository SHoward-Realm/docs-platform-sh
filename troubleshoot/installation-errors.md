# Installation Errors

While our [install script](../getting-started/install-realm-object-server/) typically provides a smooth install experience, we have seen a few issues that can arrive depending on your operating environment.  If you continue to run into issues after going through this guide, please contact us via our [forums](https://forums.realm.io/c/rmp-feedback).  

## ros: Command not found.

 `ros: Command not found.` Indicates that your terminal is not able to locate our [CLI](../manage/command-line-interface-for-ros.md).  We find that this is most often caused by using an incompatible version of `nvm`

Before running a `ros` command try running: `nvm use 8` in your terminal.  

## Trouble running \`npm install\` from your ROS directory after a \`ros init\` 

We have seen users experience installation errors when trying to install packages \(like our [graphQL adapter](../develop/integration/web-integration.md)\) inside of their ROS directories which are created by `ros init`.  Errors will typically look like: 

```text
npm ERR! path <path>/node_modules/realm-object-server/node_modules/abbrev
npm ERR! code ENOENT
npm ERR! errno -2
npm ERR! syscall rename
npm ERR! enoent ENOENT: no such file or directory, rename '<path>/node_modules/realm-object-server/node_modules/abbrev' -> '/home/ubuntu/sogorealm/sogo/node_modules/realm-object-server/node_modules/.abbrev.DELETE'
npm ERR! enoent This is related to npm not being able to find a file.
npm ERR! enoent 
```

While we plan to fix this soon, we do have a workaround.  Simply, remove the `package-lock.json` file and you will be able to install the packages.  



