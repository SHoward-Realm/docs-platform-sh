# Installation Errors

While our [install script](../installation/) typically provides a smooth install experience, we have seen a few issues that can arrive depending on your operating environment.  If you continue to run into issues after going through this guide, please contact us via our [forums](https://forums.realm.io/c/rmp-feedback).  

## ros: Command not found.

 `ros: Command not found.` Indicates that your terminal is not able to locate our [CLI](../manage/command-line-interface-for-ros.md).  We find that this is most often caused by using an incompatible version of `nvm`

Before running a `ros` command try running: `nvm use 8` in your terminal.  

## error: \[sync\] Your feature token does not allow synchronization

Starting with the release of Realm Object Server 3.5.0, a feature token is required for operation.  Depending on when your feature token was generated, it may not be compatibly with the new tokening system.  If you receive this error, please [contact us](https://support.realm.io/support/home), and we will generate a new token for your usage.  

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 

