---
description: >-
  This adds functionality to our ToDo app example showing how to use custom
  queries to explicitly control what data are synced to mobile clients
---

# Step 2- Adding Partial Sync

{% hint style="danger" %}
## _This tutorial uses functionality not yet deployed on Realm Cloud._

This tutorial uses a beta version of new Realm APIs. This code is _**not yet production ready**_ and may not reflect the final API. We will be updating this tutorial as the API and the functionality are made ready for general availability.
{% endhint %}

{% hint style="info" %}
Want to get started right away with the complete source code? Check out [https://github.com/realm/my-first-realm-app/](https://github.com/realm/my-first-realm-app/) the ios/PartialSync directory has the complete, ready-to-compile source code.
{% endhint %}

## Prerequisites {#prerequisites}

Before we get started we need a few things setup; the prerequisites for this project are:

*  [Xcode 9.0](https://itunes.apple.com/us/app/xcode/id497799835?mt=12) or later
* CocoaPods v1.2.x+
* The Realm Cloud instance URL that was generated when you created your instance \(it can be found by logging in to [https://cloud.realm.io](https://cloud.realm.io/), and clicking the `Copy Instance URL` link\).

This tutorial uses CocoaPods to install the required Realm frameworks. You can install Cocoapods by [following the installation instructions](https://guides.cocoapods.org/using/getting-started.html) at cocoapods.org.

## Step 1: Download the completed iOS ToDo Basic Tutorial {#step-1:-download-the-completed-ios-todo-basic-tutorial}

Rather than start from a clean slate, we will build on top of the iOS ToDo app we presented in [Step 1- My First Realm App](step-1-my-first-realm-app.md). This repository contains both iOS and Android source code; we will be working with only the iOS code for this tutorial.

1. Open a new Terminal window
2. Change directory to a convenient location to keep this source code \(e.g., `cd ~/Desktop`\)
3. Download the iOS App Tutorial source code with the command git command:

   \`git clone [https://github.com/realm/my-first-realm-app.git\`](https://github.com/realm/my-first-realm-app.git) 

4. Change into the downloaded directory with `cd my-first-app`
5. Remove the Android source code and git control files with `rm -rf android .git`
6. Change directory into the ios directory with `cd ios`
7. Change directory into the SyncIntro directory with `cd SyncIntro`

> Note: there is a _PartialSync_ folder here as well - in this tutorial we will be adding to the basic sync tutorial to bring it up to the level of the completed partial sync version.

## Step 2: Update/Install the Realm SDK with Cocoapods {#step-2:-updateinstall-the-realm-sdk-with-cocoapods}

In order to be ready to add our new functionality, we need to update the Realm Framework; this is done using cocoapods.

This tutorial uses a beta version of our sync APIs that implements our query-based selective sync mechanism; the Cocoapods file will need to be updated. As shipped, the [SyncIntro](step-1-my-first-realm-app.md) code has a `Podfile` that contains the following:

{% code-tabs %}
{% code-tabs-item title="Podfile" %}
```ruby
# Uncomment the next line to define a global platform for your project
platform :ios, '11.0'

target 'ToDo' do
  # Comment the next line if you're not using Swift and don't want to use dynamic frameworks
  use_frameworks!
  pod 'RealmSwift'
  # Pods for ToDo

end
```
{% endcode-tabs-item %}
{% endcode-tabs %}

You will need to edit this \(which can be done either in Xcode by opening the `Podfile` in the `Pods` section of the file navigator, or with your favorite text editor.

The line that references `pod 'RealmSwift'` will need to be deleted and replaced with the following:

{% code-tabs %}
{% code-tabs-item title="Podfile" %}
```ruby
pod 'RealmSwift', '~> 3.2.0-beta.1'
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Once the `Realm` and `RealmSwift` lines have been updated, save your changes. Then, In your terminal window, in the `SyncIntro` folder execute the command:

`pod update --repo-update`

This will download the Realm frameworks and update the Xcode workspace file. Once this process completes you can then open the iOS Project workspace file with Xcode, it is named `ToDo.xcworkspace`.

## Step 3: Setting the Realm Cloud Instance URL {#step-3:-setting-the-realm-cloud-instance-url}

Locate and open the `Constants.swift` file in the Xcode file navigator; replace the two instances of `MY_INSTANCE_ADDRESS` with the hostname portion of the Realm Cloud instance you copied from the Realm Cloud Portal \(e.g., mycoolapp.us1.cloud.realm.io\).

{% code-tabs %}
{% code-tabs-item title="Constants.swift" %}
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

## Step 4: Adding the Project Model {#step-4:-adding-the-project-model}

Now that we have told the project what server to connect to, we can proceed with adding new models and functionality to the application. Let's start by creating a new Swift file called `Project.swift` . This will be the model that represents a grouped collection of ToDo items.

> Creating a new Model file is done by selecting `File > New > File..`. and selecting "Swift File" from the file type selector panel. Press `next` then enter `Project` for the file name \(the `.swift` extension will be added automatically\) and navigate, if needed, to the Xcode project directory for `SyncIntro` and save the file.

Add the following code into the new file to complete the creation of the Project model.

{% code-tabs %}
{% code-tabs-item title="Project.swift" %}
```swift
import RealmSwift

class Project: Object {
    @objc dynamic var projectId: String = UUID().uuidString
    @objc dynamic var owner: String = ""
    @objc dynamic var name: String = ""
    @objc dynamic var timestamp: Date = Date()

    let items = List<Item>()
    
    override static func primaryKey() -> String? {
        return "projectId"
    }   
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Step 5: Adding a Project Lists View Controller {#step-5:-adding-a-project-lists-view-controller}

Now that we have a model to work with, we can create a view controller in which we can both create new projects or view/select existing ones.

> As you did with the `Project` model , creating a new view controller is done by selecting `File > New > File..`. and this time selecting "Cocoa Touch Class" from the file type selector panel. Press `next` then enter ProjectsViewController for the file name \(the `.swift` extension will be added automatically\) and navigate, if needed, to your Xcode project directory and save the file...

Before we add the entire contents needed for this file we will look at the relevant code fragments that make this different from the previous simple full-synced version of the ToDo list app:

**Opening the Realm with Partial Sync**:

```swift
let syncConfig = SyncConfiguration(user: SyncUser.current!, realmURL: Constants.REALM_URL, isPartial: true)
```

The important change here is an additional parameter applied to the Realm SyncConfiguration that tells the server not to download all of the Realm's data on open/connection but rather to only respond to the client's specific subscription request, based on a specific query.

**Setting up the Sync Query**:

```swift
projects = realm.objects(Project.self).filter(NSPredicate(format: "owner = '\(SyncUser.current!.identity!)'")).sorted(byKeyPath: "timestamp", ascending: false)
```

Here the syntax we used with a fully-synced Realm and partial sync are the same - the difference is with a _with a fully synced Realm the data are already synchronized_ \(or may be in the process of being downloaded\). The query is selecting `Project` model records where the `owner` property matches the ID of the currently logged in user \(`SyncUser.current!.identity!`\), and these will be sorted in date order, newest first.

It's important to note that with partial sync, _no data are synchronized from the server to the client until a subscription is presented._ In our application the presentation of the subscription is done just as we are preparing to display the data in the controller's `viewDidLoad` method as follows:

```swift
subscription = projects.subscribe(named: "my-projects")
// here you might show an activity spinner to indicate you 
// are waiting for the subscription to be processed
subscriptionToken = subscription.observe(\.state, options: .initial) { state in
    if state == .complete {
       // here you might remove any activity spinner
       }
}
```

The first line is the subscription itself. You can have any number of subscriptions for different kinds of data depending on your application's requirements.

The second line is a subscription token - an observer pattern used frequently in Realm to allow applications to react to changes in the state of Realm objects. It is very similar to the `NotificationToken` used to respond changes in result sets for Realm queries; the subscription token lets your application react to changes in the status of the fulfillment of your subscriptions.

The rest of this class is more or less the same as the fully-synced `ItemsViewController` in the intro version of this application where the goal was just to build a simple list of ToDo items, except it allows the user to create and persist new `Project` records.

**Passing the Project to the ItemsViewController**:

Whether we have 1 or more existing Projects or are just creating one, we will want to be able to add items to that project by selecting it in this view. This is done by passing a project record to the ItemsViewController so it can find any items \(if any\) associated with our Project.

In the `didSelectRow:indexPath` method we set up the next view controller as follows:

```swift
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
   let project = projects[indexPath.row]
   let itemsVC = ItemsViewController()
   itemsVC.project = project
   self.navigationController?.pushViewController(itemsVC, animated: true)
    }
```

This gets the project we selected from the list of projects we own that were returned as part of the subscription. It then creates an instance of the `ItemsViewController` and passes a copy of the `Project` record into the new controller; we then push the new view controller onto the navigation controller stack which causes it to be displayed.

Now that we have the important concepts covered, add the following code to your new view controller file:

{% code-tabs %}
{% code-tabs-item title="ProjectViewController.swift" %}
```swift
import UIKit
import RealmSwift

class ProjectsViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
    
    let realm: Realm
    let projects: Results<Project>
    var notificationToken: NotificationToken?
    var subscriptionToken: NotificationToken?
    var subscription: SyncSubscription<Project>!

    
    var tableView = UITableView()
    let activityIndicator = UIActivityIndicatorView()
    
    override init(nibName nibNameOrNil: String?, bundle nibBundleOrNil: Bundle?) {
        let syncConfig = SyncConfiguration(user: SyncUser.current!, realmURL: Constants.REALM_URL, isPartial: true)
        realm = try! Realm(configuration: Realm.Configuration(syncConfiguration: syncConfig))
        
        projects = realm.objects(Project.self).filter(NSPredicate(format: "owner = '\(SyncUser.current!.identity!)'")).sorted(byKeyPath: "timestamp", ascending: false)
        
        super.init(nibName: nil, bundle: nil)
    }
    
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "My Projects"
        view.addSubview(tableView)
        view.addSubview(activityIndicator)
        activityIndicator.center = self.view.center
        activityIndicator.color = .darkGray
        activityIndicator.isHidden = false
        activityIndicator.hidesWhenStopped = true
        
        tableView.frame = self.view.frame
        tableView.delegate = self
        tableView.dataSource = self
        
        navigationItem.leftBarButtonItem = UIBarButtonItem(barButtonSystemItem: .add, target: self, action: #selector(addItemButtonDidClick))
        navigationItem.rightBarButtonItem = UIBarButtonItem(title: "Logout", style: .plain, target: self, action: #selector(logoutButtonDidClick))
        
        // In a Partial Sync use case this is where we tell the server we want to
        // subscribe to a particular query.
        subscription = projects.subscribe(named: "my-projects")
        
        activityIndicator.startAnimating()
        subscriptionToken = subscription.observe(\.state, options: .initial) { state in
            if state == .complete {
                self.activityIndicator.stopAnimating()
            } else {
                print("Subscription State: \(state)")
            }
        }
        
        
        notificationToken = projects.observe { [weak self] (changes) in
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
    }
    
    deinit {
        notificationToken?.invalidate()
        subscriptionToken?.invalidate()
        activityIndicator.stopAnimating()
    }
    
    @objc func addItemButtonDidClick() {
        let alertController = UIAlertController(title: "Add New Project", message: "", preferredStyle: .alert)
        
        alertController.addAction(UIAlertAction(title: "Save", style: .default, handler: {
            alert -> Void in
            let textField = alertController.textFields![0] as UITextField
            let project = Project()
            project.name = textField.text ?? ""
            project.owner = SyncUser.current!.identity!
            try! self.realm.write {
                self.realm.add(project)
            }
            // do something with textField
        }))
        alertController.addAction(UIAlertAction(title: "Cancel", style: .cancel, handler: nil))
        alertController.addTextField(configurationHandler: {(textField : UITextField!) -> Void in
            textField.placeholder = "New Item Text"
        })
        self.present(alertController, animated: true, completion: nil)
    }
    
    @objc func logoutButtonDidClick() {
        let alertController = UIAlertController(title: "Logout", message: "", preferredStyle: .alert)
        alertController.addAction(UIAlertAction(title: "Yes, Logout", style: .destructive, handler: {
            alert -> Void in
            SyncUser.current?.logOut()
            self.navigationController?.setViewControllers([WelcomeViewController()], animated: true)
        }))
        alertController.addAction(UIAlertAction(title: "Cancel", style: .cancel, handler: nil))
        self.present(alertController, animated: true, completion: nil)
    }
    
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return projects.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "Cell") ?? UITableViewCell(style: .subtitle, reuseIdentifier: "Cell")
        cell.selectionStyle = .none
        let project = projects[indexPath.row]
        cell.textLabel?.text = project.name
        cell.detailTextLabel?.text = project.items.count > 0 ? "\(project.items.count) task(s)" : "No tasks"
        return cell
    }
    
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        let project = projects[indexPath.row]
        let itemsVC = ItemsViewController()
        itemsVC.project = project
        self.navigationController?.pushViewController(itemsVC, animated: true)
    }
    
    func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCellEditingStyle, forRowAt indexPath: IndexPath) {
        guard editingStyle == .delete else { return }
        let project = projects[indexPath.row]
        if project.items.count > 0 {
            confirmDeleteProjectAndTasks(project: project)
        } else {
            deleteProject(project)
        }
    }
    
    @objc func confirmDeleteProjectAndTasks(project: Project) {
        let alertController = UIAlertController(title: "Delete \(project.name)?", message: "This will delete \(project.items.count) task(s)", preferredStyle: .alert)
        alertController.addAction(UIAlertAction(title: "Yes, Delete \(project.name)", style: .destructive, handler: {
            alert -> Void in
            self.deleteProject(project)
        }))
        alertController.addAction(UIAlertAction(title: "Cancel", style: .cancel, handler: nil))
        self.present(alertController, animated: true, completion: nil)
    }

    func deleteProject(_ project:Project) {
        try! realm.write {
            realm.delete(project.items)
            realm.delete(project)
        }
    }
    
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Step 6: Updating the WelcomeViewController {#step-6:-updating-the-welcomeviewcontroller}

There is one more change we have to make to allow this new `ProjectsViewController` to be the focus of our enhanced ToDo App: make `ProjectsViewController` the controller we present after logging in.

To do this, locate and open the `WelcomeViewController` in the file navigator. First let's examine the original code:

```swift
SyncUser.logIn(with: creds, server: Constants.AUTH_URL, onCompletion: { [weak self](user, err) in
    if let _ = user {
        self?.navigationController?.viewControllers = [ItemsViewController()]  
    } else if let error = err {
        fatalError(error.localizedDescription)
    }
})
```

At line 3, after successfully logging in we go to our list of ToDo items. And now the updated version:

In this version we are going to _reset the default view_ for the Navigation Controller to be our list of Projects.

```swift
SyncUser.logIn(with: creds, server: Constants.AUTH_URL, onCompletion: { [weak self](user, err) in
    if let _ = user {
        self?.navigationController?.viewControllers = [ProjectsViewController()]
    } else if let error = err {
        fatalError(error.localizedDescription)
    }
})
```

## Recap: What We've Done So Far... {#recap:-what-we've-done-so-far...}

The big difference between the simple version of the application where we created a simple ToDo list and the version we are creating here is the idea of a project as a container for collections of ToDo items. We selectively subscribe to `Project` records in order to get only the data we need, not all the possible data that may exist inside a Realm.

Let's recap what we have done so far; in our partial sync powered ToDo app, we...

1. Allow the user to log in with a nick name, but instead of jumping right to a list to the ToDo's in the ItemsViewController we present a ProjectsViewController.
2. Allow the user to see a list of their existing projects, or to create one or more new projects that act a grouping mechanism \(via the `ProjectsViewController`\) for TODo items. We know about \(can find\) these projects because of the ability to _subscribe_ to `Project` records that _match a specific query_ we are interested in. In this case projects that were created by our own user - represented by the `SyncUser.current.identity` which is Realm's way of tracking the currently logged in user.
3. Have set up the `ProjectsViewController` in such a way that when we tap on the row for a given project, that project record is passed over to the ItemsViewController which uses this information to show us any existing ToDo items and/or create new ones that will be added to the selected project.

## Step 7: Connecting ToDo Items to a Project {#step-7:-connecting-todo-items-to-a-project}

The existing ItemsViewController created as part of the SyncIntro contains about 99% of the code we need in order to complete this project; the additions and changes we need to make are truly minimal.

### Using the Project Record {#using-the-project-record}

In the previous section we passed in the project record for the row the user taps in the `ProjectsViewController`; near the top of the file where the Realm and results variables are declared, change `items` type to be `List<Item>?` and add a new variable called optional `project` variable as shown below:

```swift
var items: List<Item>?
var project: Project?
```

Next we are going to use the `Project` object that was passed in to get the list \(if any\) of existing items for this project. In the viewDidLoad method, just after the `super.viewDidLoad()` call, add the following line:

```swift
self.items = project?.items
```

The change in type for the `items` variable is so that it matches the type declared in the Project model and we don't have to re-cast anything.

### Adding new ToDo Items to the Project {#adding-new-todo-items-to-the-project}

Adding new to do Items works exactly as it did in the non-partial sync version; the difference becomes clear a the point where we save the new ToDo item. Rather than add the new item to the Realm directly, we append the new item to the list of items managed by the `Project`. This is accomplished by changing the addition code \(in the method `addItemButtonDidClick`\) :

The non-partial sync version writes to the Realm using a local instance of the Realm which is created at the time the class was loaded \(lines 7-8\):

```swift
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
```

Here's the updated version. Notice the difference? The partial sync version uses the Realm instance attached to the project record like this:

```swift
alertController.addAction(UIAlertAction(title: "Save", style: .default, handler: {
    alert -> Void in
    let textField = alertController.textFields![0] as UITextField
    let item = Item()
    item.body = textField.text ?? ""

    try! self.project?.realm?.write {
        self.project?.items.append(item)
    }
    // do something with textField
}))
```

Whenever we need to write or update a record we do the same thing - use the reference to the Realm that is held by the project the user selected and we passed into the `ItemsViewController`: `self.project?.realm` Now that we have the basics down, we can replace the all of the references to the local Realm in the other table view methods that support editing or deleting ToDo Item rows:

```swift
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    let item = items?[indexPath.row]
    try! self.project?.realm?.write {
        item!.isDone = !(item!.isDone)
    }
}
    
func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCellEditingStyle, forRowAt indexPath: IndexPath) {
    guard editingStyle == .delete else { return }
    let item = items?[indexPath.row]
    try! self.project?.realm?.write {
        self.project?.realm?.delete(item!)
    }
}
```

With that done, we can delete the declaration of the local Realm variable at the top of the file,

```swift
let realm: Realm
```

and the references to it in the class's init method:

```swift
let syncConfig = SyncConfiguration(user: SyncUser.current!, realmURL: Constants.REALM_URL)
self.realm = try! Realm(configuration: Realm.Configuration(syncConfiguration: syncConfig))
```

You should now be able to build and run the application, log in and create projects and ToDo Items:

![](https://blobscdn.gitbook.com/v0/b/gitbook-28427.appspot.com/o/assets%2F-L22PAstP49i9mPl4sPN%2F-L4307kfLEbsr_KglEYD%2F-L430AZfLqJfrAVoaUQJ%2Fios-partalSync.gif?alt=media&token=2c910be8-9a55-4f4a-a4de-30903d46ad9e)

Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE) 

