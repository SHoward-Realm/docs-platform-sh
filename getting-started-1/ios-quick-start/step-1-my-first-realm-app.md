# Step 1 - My First Realm App

## Prerequisites {#prerequisites}

Before we get started we need a few things setup; the prerequisites for this project are:

*  [Xcode 9.0](https://itunes.apple.com/us/app/xcode/id497799835?mt=12) or later
* You will need your Realm Cloud instance URL that was generated when you created your instance \(it can be found by logging in to the [cloud portal](https://cloud.realm.io/), and clicking the `Copy Instance URL` link\)

![](https://blobscdn.gitbook.com/v0/b/gitbook-28427.appspot.com/o/assets%2F-L22PAstP49i9mPl4sPN%2F-L3F0A-0xyNVXJUobiJJ%2F-L3F0IqFgvrg4Lz_vbD1%2FToDo-add-task-wide-take2.gif?alt=media&token=942c81fc-1540-4fe4-abb2-429c349578ff)

This tutorial should take around 20-30 minutes

## Step 1: Create a New iOS Project {#step-1:-create-a-new-ios-project}

1. Open Xcode, create a new iOS Project \(we recommend the "Single View" application\). Let's name it "**iOSToDoApp**." When prompted, save it to a convenient place such as your desktop.

## Step 2: Initial Run Test {#step-2:-initial-run-test}

Before we dive into adding code, it's a good idea to make sure that Xcode is configured correctly and that we can successfully compile the app template we just created. In order to do this, select a simulator \(for example the iPhoneX\) from the build menu, then press the build/run icon. The app should build and launch and show the basic empty app shown here: 

![](https://blobscdn.gitbook.com/v0/b/gitbook-28427.appspot.com/o/assets%2F-L22PAstP49i9mPl4sPN%2F-L4790JN_GLg-UPTdATs%2F-L47BmQxFuJuoorLa7J_%2FSyncIntro-TemplateBuild.gif?alt=media&token=9659bdee-60b7-4324-8cc1-23d16c5f8894)

## Step 3: Installing the Realm Frameworks {#step-3:-installing-the-realm-frameworks}

To use Realm Cloud in your iOS app, you'll need to add the `Realm` and `RealmSwift` frameworks to your project. This is most easily accomplished using using either of the major iOS dependency managers: [Cocoapods](https://cocoapods.org/) or [Carthage](https://github.com/Carthage/Carthage/blob/master/README.md). Alternatively, you can install the framework by downloading it directly from Realm and dragging it into your Xcode project. The tabs below give detailed instructions on how to install the Realm framework for each of these options.

{% tabs %}
{% tab title=".Cocoapods" %}
{% hint style="warning" %}
Make sure you have Cocoapods version 1.2.x or higher. 

If you run into any errors or if you need to install Cocoapods, please [follow the installation instructions](https://guides.cocoapods.org/using/getting-started.html) at [Cocoapods.org](https://cocoapods.org/).
{% endhint %}

If you choose to use Cocopods, _you will need to close your Xcode project for the next steps_; we will re-open it once Cocopods is set up.

Cocoapods uses a `Podfile` to track, load, and manage your project's external dependencies; to create the `Podfile`, open a new terminal window, change the directory to your newly created Xcode project and type

```text
pod init
```

This will create a new `Podfile` - open this file with an editor of your choice and look for the section that starts with `# Pods for iOSToDoApp`, then add the following line:

```text
pod 'RealmSwift'
```

Save your changes, and then run the following command from the terminal window:

```text
pod install --repo-update
```

This will download all the required packages and create a new Xcode workspace for your project. Once the Cocoapods installation process completes open the new `iOSToDoApp.xcworkspace` to continue to work on your project.
{% endtab %}

{% tab title="Carthage" %}
{% hint style="warning" %}
Make sure you have Carthage version 0.17.x or higher.
{% endhint %}

If you prefer [Carthage](https://github.com/Carthage/Carthage#installing-carthage), you can add the following to your `Cartfile`**.** You will need to create the `Cartfile` in your Xcode project directory, then add the following line:

```text
github "realm/realm-cocoa"
```

Save your changes and then download and build the packages with this command in the terminal window:

```text
carthage bootstrap
```

{% hint style="warning" %}
This process can take 10 minutes to complete and depending on your installed version of Xcode, you may see warning about compiler incompatibilities; these can be ignored.
{% endhint %}

After Carthage builds the Realm frameworks, drag `RealmSwift.framework` and `Realm.framework` files from the Carthage/Build/iOS directory to the “Linked Frameworks and Libraries” section of your Xcode project’s “General” settings tab. Click OK when prompted to copy the framework files.

In your application target’s **“Build Phases”** settings tab, click the **+** icon and choose **“New Run Script Phase”**. Create a Run Script with the following contents:

```text
/usr/local/bin/carthage copy-frameworks
```

and add the paths to the frameworks under **“Input Files”**

```text
$(SRCROOT)/Carthage/Build/iOS/Realm.framework$(SRCROOT)/Carthage/Build/iOS/RealmSwift.framework
```
{% endtab %}

{% tab title="Direct Download" %}
In order to load the framework directly, you will need to [download the latest version of the Realm framework from this link](https://static.realm.io/downloads/swift/realm-swift-3.1.0.zip?_ga=2.186413289.1750353362.1516990351-225787661.1516643938). Once the download has completed there will be a new folder in your Downloads directory named `Realm-swift-x.y.z` \(where x.y.z is a version number\). Open this folder, navigate to the latest swift version \(at the time of this writing 4.02\) and drag the `Realm.Framework` and `RealmSwift.Framework` file into the Frameworks and Libraries tab as shown here. 

![](https://blobscdn.gitbook.com/v0/b/gitbook-28427.appspot.com/o/assets%2F-L22PAstP49i9mPl4sPN%2F-L489y7tLIiU8JIfSMmE%2F-L48AK9uLRIGaZS8hVCC%2FiosToDo-install-realm-frameworks.gif?alt=media&token=9bc7cea9-2b37-406e-87d9-d9a192be0a9d)
{% endtab %}
{% endtabs %}

### Testing the Framework Build {#testing-the-framework-build}

{% hint style="danger" %}
Reminder: If you are using Cocoapods, you will need to close the Xcode `iOSToDoApp.xcproj` \(which is the default project made when you created your app\) and reopen the newly-created `iOSToDoApp.xcworkspace` to be able to successfully compile with the Realm framework.
{% endhint %}

At this point you will want to re-run the template project to ensure that the addition of the Realm Frameworks was succesful. You won't see any changes in the app display in the simulator, but the app should compile without error.

## Step 4: Remove Unecessary Files {#step-4:-remove-unecessary-files}

The default app template contains a few files we won't need in this example, so we'll remove them to reduce any possible confusion.

1. Open the `Info.plist` for your target \(this will be in the iOSToDoApp group in the file navigator\)
2. Find the `Main storyboard file base name` entry and remove it by pressing on the `-` icon next to it in the property editor
3. Find the `Main.storyboard` file in Xcode's file navigator and delete it by selecting it and then pressing the `Delete` key

Edit the **AppDelegate.swift** file and replace the `func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?)` method to match the following code below. This sets up `UINavigationController` with a `WelcomeViewController` which we will create in an upcoming step.

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
    // Override point for customization after application launch.
    
    window = UIWindow(frame: UIScreen.main.bounds)
    window?.makeKeyAndVisible()
    window?.rootViewController = UINavigationController(rootViewController: WelcomeViewController())
    
    return true
}
```

## Step 5: Create a Constants file and Set the Realm Cloud Instance URL {#step-5:-create-a-constants-file-and-set-the-realm-cloud-instance-url}

Next we will create a new Swift file to contain our app's constants. This is done by selecting `File > New > File..`. in Xcode and then selecting "Swift File" from the file type selector panel. Press `next` then enter `Constants` for the file name \(the `.swift` extension will be added automatically\) and navigate, if needed, to your Xcode project directory and save the file.

Paste the code snippet here into the newly created `Constants.swift`; replace the string  `MY_INSTANCE_ADDRESS` with the hostname portion of the Realm Cloud instance you copied from the Realm Cloud Portal \(e.g., `mycoolapp.us1.cloud.realm.io`\). We will use these constants \(e.g. `Constants.AUTH_URL` and `Constants.REALM_URL`_\)_ wherever we need to references our Realm instance.

{% hint style="warning" %}
**NOTE**: The Realm Cloud Portal presents fully specified URLs \(e.g., _https:_//_appname.cloud.realm.io_\); be sure to paste in only the host name part \(e.g., _appname.cloud.realm.io_\) into your copy of the `Constants.swift` file.
{% endhint %}

{% hint style="warning" %}
**Self-Hosted:** The code snippet below is optimized for cloud. When using a self-hosted version of Realm Object Server, directly set the `AUTH_URL` and `REALM_URL` variables. _It is likely you won't initially have SSL/TLS setup, so be careful with `http[s]` and `realm[s]`_.
{% endhint %}

{% code-tabs %}
{% code-tabs-item title="constants.swift" %}
```swift
import Foundation
struct Constants {
    // **** Realm Cloud Users:
    // **** Replace MY_INSTANCE_ADDRESS with the hostname of your cloud instance
    // **** e.g., "mycoolapp.us1.cloud.realm.io"
    // ****
    // ****
    // **** ROS On-Premises Users
    // **** Replace the AUTH_URL and REALM_URL strings with the fully qualified versions of
    // **** address of your ROS server, e.g.: "http://127.0.0.1:9080" and "realm://127.0.0.1:9080"
    
    static let MY_INSTANCE_ADDRESS = "MY_INSTANCE_ADDRESS" // <- update this
    
    static let AUTH_URL  = URL(string: "https://\(MY_INSTANCE_ADDRESS)")!
    static let REALM_URL = URL(string: "realms://\(MY_INSTANCE_ADDRESS)/ToDo")!
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Step 6: Add WelcomeViewController and Authentication Dialog {#step-6:-add-welcomeviewcontroller-and-authentication-dialog}

Create a new view controller called `WelcomeViewController.swift`. This will be the controller you see when the app starts up and will be used to log in to the app and connect your application to the Realm Cloud.

Creating a new view controller is done by selecting `File > New > File..`. in Xcode and then selecting "Cocoa Touch Class" from the file type selector panel. Press `next` then enter `WelcomeViewController` for the class name and `UIViewController` for the subclass name. Press `next` again, and when prompted navigate, if needed, to your Xcode project directory and save the file.

Near the top of the new view controller file, below the existing import statement, import the Realm framework by adding this line:

```text
import RealmSwift
```

{% hint style="warning" %}
Xcode may show errors next to the new import line or other lines; this is normal and these will disappear as we add code and the app is compiled.
{% endhint %}

Next, we will add a `viewDidAppear` method of the view controller with the following snippet. The main function of this new method is to log a user in to your Realm Cloud instance. We are just going to login using a "nickname" you provide from an dialog that is presented when the app launches.

> **How the WelcomeViewController Authenticates with Realm Cloud**
>
> Realm supports a number of authentication methods: For prototyping \(or low-security applications\) the `Nickname` method - which allows you to log in without needing a password -- is very convenient, and it's what we've chosen for this short introduction. Nickname credentials are easily constructed with: `let creds = SyncCredentials.nickname("sally", isAdmin: true)`
>
> You’ll notice that `isAdmin` is set to true; this allows this user full control over the Realm. We will revisit this later, but for the purposes of getting started quickly we recommend that you keep this to true to encounter less friction when learning more about Realm.
>
> **Note**: In a production setting you would need to turn this provider off in to production so that random strangers cannot login.

The `viewDidAppear` method does the initial display of or WelcomeViewController: Add this new method after the closing brace of the stub `viewDidLoad()` method near the top of the file:

```swift
override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
    title = "Welcome"
        
    if let _ = SyncUser.current {
        // We have already logged in here!
        self.navigationController?.pushViewController(ItemsViewController(), animated: true)
     } else {
          let alertController = UIAlertController(title: "Login to Realm Cloud", message: "Supply a nice nickname!", preferredStyle: .alert)
            
         alertController.addAction(UIAlertAction(title: "Login", style: .default, handler: { [unowned self]
          alert -> Void in
          let textField = alertController.textFields![0] as UITextField
          let creds = SyncCredentials.nickname(textField.text!, isAdmin: true)
                
           SyncUser.logIn(with: creds, server: Constants.AUTH_URL, onCompletion: { [weak self](user, err) in
            if let _ = user {
                    self?.navigationController?.pushViewController(ItemsViewController(), animated: true)
              } else if let error = err {
                     fatalError(error.localizedDescription)
                    }
                })
            }))
            alertController.addAction(UIAlertAction(title: "Cancel", style: .cancel, handler: nil))
            alertController.addTextField(configurationHandler: {(textField : UITextField!) -> Void in
                textField.placeholder = "A Name for your user"
            })
            self.present(alertController, animated: true, completion: nil)
        }
    }
```

The bulk of this code sets up your user's credential , connects to the Realm Cloud. If the login is successful it then transitions to an `ItemsViewController`, which we will build in the next several steps. In the event a login attempt fails, this method will also post a dialog with a Realm error message detailing what went wrong.

## Step 7: Adding the Realm Model for Item {#step-7:-adding-the-realm-model-for-item}

Before we get to the creation of the ToDo list in `ItemsViewController` we need to define the Realm model that will describe our ToDo list items. Create a new file called `Item.swift` and add in the following model definition

As you did with the `WelcomeViewController` before, creating a new view controller is done by selecting `File > New > File..`. and this time selecting "Swift File" from the file type selector panel. Press `next` then enter `Item` for the file name \(the `.swift` extension will be added automatically\) and navigate if needed to your Xcode project directory and save the file:

```swift
import RealmSwift

class Item: Object {
    
    @objc dynamic var itemId: String = UUID().uuidString
    @objc dynamic var body: String = ""
    @objc dynamic var isDone: Bool = false
    @objc dynamic var timestamp: Date = Date()
    
    override static func primaryKey() -> String? {
        return "itemId"
    }
    
}
```

This is a representation of a Realm Model that will hold each task of the ToDo list. All properties are required and have default values. We will use the `timestamp` property to sort the collection of Items.

## Step 8: Add an ItemsViewController  {#step-8:-add-an-itemsviewcontroller}

Create a new ViewController called `ItemsViewController.swift` and import `RealmSwift`

As before, creating a new view controller is done by selecting `File > New > File..`. and then selecting "Cocoa Touch Class" from the file type selector panel. Press `next` then enter `ItemsViewController` for the class name and `UIViewController` for the subclass name. Press `next` again, and when prompted navigate if needed to your Xcode project directory and save the file. As with the `WelcomeViewController`, import RealmSwift near the top of the file:

```swift
import UIKit
import RealmSwift // <- Insert this 

class ItemsViewController: UIViewController {
// ... template code here ... 
}
```

### Add Instance Variables for the Realm and Items {#add-instance-variables-for-the-realm-and-items}

Create two instance variables just after the line declaring the `ItemsViewController` class:

```swift
class ItemsViewController: UIViewController {
    let realm: Realm            // <- Insert this
    let items: Results<Item>    // <- Insert this
```

Next, initialize them in the class's constructor; copy this code snippet in place just after the instance variable declaration:

```swift
override init(nibName nibNameOrNil: String?, bundle nibBundleOrNil: Bundle?) {
    let syncConfig = SyncConfiguration(user: SyncUser.current!, realmURL: Constants.REALM_URL)
    self.realm = try! Realm(configuration: Realm.Configuration(syncConfiguration: syncConfig, objectTypes:[Item.self]))
    self.items = realm.objects(Item.self).sorted(byKeyPath: "timestamp", ascending: false)
    super.init(nibName: nil, bundle: nil)
}

required init?(coder aDecoder: NSCoder) {
    fatalError("init(coder:) has not been implemented")
}
```

Notice that we have created a synced Realm pointing to your Realm Cloud instance. In addition we've created a query to return a list of Items.

### Testing Authentication {#testing-authentication}

We've added a lot to this application, but so far haven't had an opportunity to test and and see if the work we've done so far works. Let's add just a few more lines of code so that we can build and run and ensure we're still on track.

Near the top of our class file, right after the end of the init method you just added we will add to the existing `viewDidLoad` method. This method sets up the view controller just after it is initialized. We will add more setup instructions shortly, but for now, let's just create a button handler that will add a logout button. This will allow us to test authentication and ensure that we can connect to our Realm Cloud instance and login. Once this is ready we can add the rest of the support for adding and managing ToDo items.

```swift
override func viewDidLoad() {
super.viewDidLoad()

navigationItem.rightBarButtonItem = UIBarButtonItem(title: "Logout", style: .plain, target: self, action: #selector(rightBarButtonDidClick))
}
```

Once this is in place we need to add a method that actually performs the logout function. This is done with a button handler. Add the method below to your view controller class -- it can be anywhere in the file; we'd suggest adding it right after the closing brace of the `viewDidLoad` method:

```swift
@objc func rightBarButtonDidClick() {
    let alertController = UIAlertController(title: "Logout", message: "", preferredStyle: .alert)
    alertController.addAction(UIAlertAction(title: "Yes, Logout", style: .destructive, handler: {
        alert -> Void in
        SyncUser.current?.logOut()
        self.navigationController?.setViewControllers([WelcomeViewController()], animated: true)
    }))
    alertController.addAction(UIAlertAction(title: "Cancel", style: .cancel, handler: nil))
    self.present(alertController, animated: true, completion: nil)
    }
```

Now we can fire up the application and see if we can log in. In the Xcode toolbar press build and run the app in the simulator again. You'll see the app start up and you will be presented with an login dialog much like the the one shown below: 

![](https://blobscdn.gitbook.com/v0/b/gitbook-28427.appspot.com/o/assets%2F-L22PAstP49i9mPl4sPN%2F-L47IUOZAQ7YeimJ_O1-%2F-L47Q0rP13OKGdhFX1Qr%2FSyncIntro-AuthTest.gif?alt=media&token=ee09fd09-5922-450f-98b0-0ec6ec345642)

From here you can log in, and account will be created on the server. Press the `logout` button to log out. You can repeat this and create as many accounts on your Realm Cloud instance as you like.

With the login/logout capability ready we can move on to the main event in our application: creation and management of our ToDo list.

### Add a UITableView {#add-a-uitableview}

First we'll add an instance variable to the class to hold our table view; add this just below the other variables you created earlier near the top of the file after the class declaration:

```swift
let tableView = UITableView()
```

Next, we'll initialize the the rest of the view, adding in our `UITableView` and setting the view's title as well; to do this modify the viewDidLoad method you added earlier:

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    navigationItem.rightBarButtonItem = UIBarButtonItem(title: "Logout", style: .plain, target: self, action: #selector(rightBarButtonDidClick))
    title = "To Do Item"
    tableView.dataSource = self
    tableView.delegate = self    
    view.addSubview(tableView)
    tableView.frame = self.view.frame
}
```

In order to actually process changes to the table view we will need to subscribe to the `UITableViewDelegate` and `UITableViewDataSource` protocols. We will add these to our `ItemsViewController`just after the `UIViewController` subclass definition; the resulting declaration should look like this:

```swift
class ItemsViewController: UIViewController, UITableViewDelegate, UITableViewDataSource
```

By declaring these protocols, we must now make sure the `ItemsViewController` conforms to them.

#### Implementing UITableViewDataSource {#implementing-uitableviewdatasource}

Realm naturally works with `UITableViewDataSource` . With an ordered collection of `Results<Item>` we can always see the items in the order they were created. We will begin by implementing what each cell in the `UITableView` will look like in the `cellForRowAt` method.

Place this method after the end of the `ViewDidLoad` method:

```swift
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: "Cell") ?? UITableViewCell(style: .default, reuseIdentifier: "Cell")
    cell.selectionStyle = .none
    let item = items[indexPath.row]
    cell.textLabel?.text = item.body
    cell.accessoryType = item.isDone ? UITableViewCellAccessoryType.checkmark : UITableViewCellAccessoryType.none
    return cell
}
```

We've set the `textLabel` to equal the `item` body. When the `isDone` is marked `true` we'll change the view to a `UITableViewCellAccessoryType.checkmark` and when false a `UITableViewCellAccessoryType.none`

Next, we need to tell `UITableViewDataSource` how many items to render; add this method after the previous one:

```swift
func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return items.count
}
```

#### Implementing UITableViewDelegate {#implementing-uitableviewdelegate}

When a user clicks on a row of the `UITableView` the `didSelectRow` method is called and we mark the item as "done" in the model which updates the database. Here too, place this method after the previous ones:

```swift
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    let item = items[indexPath.row]
    try! realm.write {
        item.isDone = !item.isDone
    }
}
```

### Implementing the Add Item Functionality  {#implementing-the-add-item-functionality}

We'll create a new button to allow the user to to add new items to the ToDo list. In the `viewDidLoad` method, just after the first button handler declaration for the logout button, add the following:

```swift
navigationItem.leftBarButtonItem = UIBarButtonItem(barButtonSystemItem: .add, target: self, action: #selector(addButtonDidClick))
```

Now we will implement the logic this button executes to create and insert new ToDo items into our database. Just like the Nickname auth in the `WelcomeViewController`, we will present a `UIAlertController` with a `textField` that will prompt for the name of the ToDo `Item.`

Add the code below after the closing brace of the `viewDidLoad` method:

```swift
@objc func addButtonDidClick() {
    let alertController = UIAlertController(title: "Add Item", message: "", preferredStyle: .alert)
    
    alertController.addAction(UIAlertAction(title: "Save", style: .default, handler: {
        alert -> Void in
        let textField = alertController.textFields![0] as UITextField
        let item = Item()
        item.body = textField.text ?? ""
        try! self.realm.write {
            self.realm.add(item)
        }
        // do something with textField
    }))
    alertController.addAction(UIAlertAction(title: "Cancel", style: .cancel, handler: nil))
    alertController.addTextField(configurationHandler: {(textField : UITextField!) -> Void in
        textField.placeholder = "New Item Text"
    })
    self.present(alertController, animated: true, completion: nil)
}
```

> How ToDo Items are Added to the Synced Realm
>
> Creating a Realm Object is quite simple in Swift. First instantiate the object: In this case it's our `Item` class that we declared earlier. We can then replace our values just like any other object and use our Realm instance to create a write transaction where we then add the `Item` to the Realm:
>
> ```swift
> let item = Item()
> item.body = textField.text ?? ""
> try! self.realm.write {
>     self.realm.add(item)
> }
> ```

### Implementing the Swipe to Delete Functionality {#implementing-the-swipe-to-delete-functionality}

Let's add another method from the `UITableViewDataSource` protocol that will allow us to respond to swipes and allow deletion of ToDo Items:

```swift
func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCellEditingStyle, forRowAt indexPath: IndexPath) {
    guard editingStyle == .delete else { return }
    let item = items[indexPath.row]
    try! realm.write {
        realm.delete(item)
    }
}
```

Here too, a Realm write transaction is used to tell our Realm of our intent to make a modification to the database - this time the deletion of a specific item.

## Step 9: Adding Reactive Functionality {#step-9:-adding-reactive-functionality}

You probably noticed that the add and delete functionality above only writes to a Realm. To make sure the `UITableView` updates in response to these changes, we will use Realm's change notification listener. Whenever an `Item` is added or deleted, we can observe these changes from the Realm and then react to update the UI.

To implement this, add an optional instance variable of type `NotificationToken?` to `ItemsViewController`right below the variable declarations you entered previously :

```swift
var notificationToken: NotificationToken?
```

And at the end of the `viewDidLoad` method you can add an notification handler:

```swift
notificationToken = items.observe { [weak self] (changes) in
    guard let tableView = self?.tableView else { return }
    switch changes {
    case .initial:
        // Results are now populated and can be accessed without blocking the UI
        tableView.reloadData()
    case .update(_, let deletions, let insertions, let modifications):
        // Query results have changed, so apply them to the UITableView
        tableView.beginUpdates()
        tableView.insertRows(at: insertions.map({ IndexPath(row: $0, section: 0) }),
                             with: .automatic)
        tableView.deleteRows(at: deletions.map({ IndexPath(row: $0, section: 0)}),
                             with: .automatic)
        tableView.reloadRows(at: modifications.map({ IndexPath(row: $0, section: 0) }),
                             with: .automatic)
        tableView.endUpdates()
    case .error(let error):
        // An error occurred while opening the Realm file on the background worker thread
        fatalError("\(error)")
    }
}
```

In this method the `changes` object is filled with fine-grained notifications about which Items were changed, including the indexes of insertions, deletions, modifications, etc.

We can use this fine-grained change information to tell our `UITableView` instance which rows to change with the `beginUpdates` and `endUpdates` methods. Now any modifications to the `Item` objects will trigger animations appropriately.

We've stored the return value of `observe` into the `notificationToken` . So long as the `notificationToken` exists -- which will be whenever the ItemsViewController is on-screen -- then the observe handler will consistently be called.

If our view controller ever goes off screen \(for example if we had other views in our application\), we would want to deallocate this notification handler. The `deinit` for the UIViewController is a convenient place to add `notificationToken?.invalidate()` method which cleans up this observer for us. Add the following `deinit` method following after the init methods near the top of the file:

```swift
deinit {
    notificationToken?.invalidate()
}
```

## Step 10: Collaborate!  {#step-10:-collaborate!}

With Xcode 9, select another simulator or attach another device and run two or more apps together. For each app choose a new new nickname for each user and observe two users simultaneously editing the same ToDo List at the same time!

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 



