# Step 2 - Adding Query-based sync

## Quick Start {#quick-start}

{% hint style="success" %}
Want to get started right away with the complete source code? [Check it out here](https://github.com/realm/my-first-realm-app/tree/master/android/PartialSync)
{% endhint %}

## Introduction {#introduction}

By default Realm's synchronization will ensure that all the data that is in the Realm in the server will be mirrored to client devices, and vice versa. This behavior is simple and works well for global shared datasets or when you have user-specific data that can be easily split into separate Realms per user. However, this behavior can be limiting when you are working with more complex shared data or if you want to control exactly how much data syncs to a client device.

Query-based synchronization is new functionality provided by Realm Cloud that allows a client device to selectively choose which subset of data from the server-side Realm it wants. This allows the client Realm to act as a dynamic cache, where it can subscribe via a query to data, in addition to unsubscribing to evict the data from the client Realm.

To illustrate the concept, let's take our `ToDo` App as an example:

Currently after you login you're presented with a list of all `Item` or Tasks. It would be nice if we can:

1. Group tasks by _project_. A project is composed by a list of tasks.
2. Work with only our projects \(i.e. projects created by our user\).

Query-based sync will allow us to synchronize only our projects via a query, while avoiding pulling projects and tasks from other users. This is technically achieved by two steps:

* Add `partialRealm` option to the `SyncConfiguration`

```java
SyncConfiguration configuration = new SyncConfiguration.Builder(
                SyncUser.currentUser(),
                REALM_BASE_URL + "/default")
                .partialRealm()
                .build();
```

Alternatively we can use `SyncConfiguration.automatic()` which achieve the same thing.

* Using async queries to subscribe to the data from the server:

```java
RealmResults<Project> myProjects = realm
                .where(Project.class)
                .equalTo("owner", SyncUser.currentUser().getIdentity())
                .findAllAsync();
```

The above query will run on the Cloud instance, then return **only** projects belonging to our user. These projects contains links to their corresponding tasks, so they will also be synced as well.

That's it! Following these two steps we enabled Query-based sync in our app.

In the following sections, we're going to walk you in detail through the modifications you need to perform, in order to transform our ToDo App.

{% hint style="info" %}
If you want to jump into the final code directly, check it out from the [repository](https://github.com/realm/my-first-realm-app/tree/master/android/PartialSync). \(don't forget to update the **Constants.java** file with the URL of your Cloud instance like we did in the first part\).
{% endhint %}

## Update build.gradle {#update-build.gradle}

Query-based sync feature is released as a snapshot for the time being. To be able to resolve the dependency, update your project's `build.gradle` to include the snapshot repository and reference the correct version.

```groovy
buildscript {

    repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.0.1'
        classpath "io.realm:realm-gradle-plugin:5.0.1"

    }
}

allprojects {
    repositories {
        google()
        jcenter()
    }
}
```

## Updating the model {#updating-the-model}

As mentioned, we want to group our tasks into projects. For this, we add a new Realm model class `Project` which contains a name and a list of all related tasks.

{% hint style="info" %}
Add the following class under `model` package name
{% endhint %}

```java
package io.realm.todo.model;

import java.util.Date;

import io.realm.RealmList;
import io.realm.RealmObject;
import io.realm.annotations.PrimaryKey;
import io.realm.annotations.Required;

public class Project extends RealmObject {
    @PrimaryKey
    @Required
    private String id;

    @Required
    private String owner;

    @Required
    private String name;

    @Required
    private Date timestamp;

    private RealmList<Item> items;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getOwner() {
        return owner;
    }

    public void setOwner(String owner) {
        this.owner = owner;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Date getTimestamp() {
        return timestamp;
    }

    public void setTimestamp(Date timestamp) {
        this.timestamp = timestamp;
    }

    public RealmList<Item> getTasks() {
        return items;
    }

    public void setTasks(RealmList<Item> items) {
        this.items = items;
    }
}
```

The `owner` property represents the id of the currently connected user. This is a way to filter our projects compared to others.

## Creating a Query-based sync configuration {#creating-a-partial-sync-configuration}

As mentioned in the introduction, we need to specify the `partialSync` option when building our `SyncConfiguration` to chose Query-based sync of the Realm. For conveniency, we set this `SyncConfiguration` as the default, so we can obtain easily a Realm instance in the app.

This is done by modifying the `WelcomeActivity` . We replace the `goToItemsActivity` method by `setUpRealmAndGoToListTaskActivity` as follow

```java
private void setUpRealmAndGoToListTaskActivity(){
        Realm.setDefaultConfiguration(SyncConfiguration.automatic());
        Intent intent = new Intent(WelcomeActivity.this, ProjectsActivity.class);
        startActivity(intent);
}
```

Furthermore there are some changes in the import section and you will have to replace `goToItemsActivity`with `setUpRealmAndGoToListTaskActivity` a few places:

```java
package io.realm.todo;

import android.animation.Animator;
import android.animation.AnimatorListenerAdapter;
import android.annotation.TargetApi;
import android.content.Intent;
import android.os.Build;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;

import io.realm.ObjectServerError;
import io.realm.Realm;
import io.realm.SyncConfiguration;
import io.realm.SyncCredentials;
import io.realm.SyncUser;

import static io.realm.todo.Constants.AUTH_URL;
import static io.realm.todo.Constants.REALM_BASE_URL;

public class WelcomeActivity extends AppCompatActivity {

    private EditText mNicknameTextView;
    private View mProgressView;
    private View mLoginFormView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_welcome);

        if (SyncUser.currentUser() != null) {
            setUpRealmAndGoToListTaskActivity();
        }

        // Set up the login form.
        mNicknameTextView = findViewById(R.id.nickname);
        Button loginButton = findViewById(R.id.login_button);
        loginButton.setOnClickListener(view -> attemptLogin());
        mLoginFormView = findViewById(R.id.login_form);
        mProgressView = findViewById(R.id.login_progress);
    }

    private void attemptLogin() {
        // Reset errors.
        mNicknameTextView.setError(null);
        // Store values at the time of the login attempt.
        String nickname = mNicknameTextView.getText().toString();
        showProgress(true);

        SyncCredentials credentials = SyncCredentials.nickname(nickname, false);
        SyncUser.loginAsync(credentials, AUTH_URL, new SyncUser.Callback<SyncUser>() {
            @Override
            public void onSuccess(SyncUser user) {
                showProgress(false);
                setUpRealmAndGoToListTaskActivity();
            }

            @Override
            public void onError(ObjectServerError error) {
                showProgress(false);
                mNicknameTextView.setError("Uh oh something went wrong! (check your logcat please)");
                mNicknameTextView.requestFocus();
                Log.e("Login error", error.toString());
            }
        });
    }

    /**
     * Shows the progress UI and hides the login form.
     */
    @TargetApi(Build.VERSION_CODES.HONEYCOMB_MR2)
    private void showProgress(final boolean show) {
        int shortAnimTime = getResources().getInteger(android.R.integer.config_shortAnimTime);
        mLoginFormView.setVisibility(show ? View.GONE : View.VISIBLE);
        mLoginFormView.animate().setDuration(shortAnimTime).alpha(
                show ? 0 : 1).setListener(new AnimatorListenerAdapter() {
            @Override
            public void onAnimationEnd(Animator animation) {
                mLoginFormView.setVisibility(show ? View.GONE : View.VISIBLE);
            }
        });
        mProgressView.setVisibility(show ? View.VISIBLE : View.GONE);
        mProgressView.animate().setDuration(shortAnimTime).alpha(
                show ? 1 : 0).setListener(new AnimatorListenerAdapter() {
            @Override
            public void onAnimationEnd(Animator animation) {
                mProgressView.setVisibility(show ? View.VISIBLE : View.GONE);
            }
        });
    }

    private void setUpRealmAndGoToListTaskActivity(){
        Realm.setDefaultConfiguration(SyncConfiguration.automatic());
        Intent intent = new Intent(WelcomeActivity.this, ProjectsActivity.class);
        startActivity(intent);
    }
}
```

## Adding ProjectsActivity {#adding-projectsactivity}

This activity is responsible of displaying the list of projects belonging to the current user. The UI part follows the same logic as in the existing `ItemsActivity` . It uses a recycler view and a [FAB](https://developer.android.com/reference/android/support/design/widget/FloatingActionButton.html) with a dialog to create new projects.

* When creating a new `Project` we set the owner to the id of the current user. This is done using

```java
String userId = SyncUser.currentUser().getIdentity();
```

The final version should look like:

```java
package io.realm.todo;

import android.content.Intent;
import android.os.Bundle;
import android.support.annotation.Nullable;
import android.support.v7.app.AlertDialog;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.widget.EditText;

import java.util.Date;
import java.util.UUID;

import io.realm.Realm;
import io.realm.RealmResults;
import io.realm.Sort;
import io.realm.SyncUser;
import io.realm.todo.model.Project;
import io.realm.todo.ui.ProjectsRecyclerAdapter;

public class ProjectsActivity extends AppCompatActivity {
    private Realm realm;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_items);

        setSupportActionBar(findViewById(R.id.toolbar));

        findViewById(R.id.fab).setOnClickListener(view -> {
            View dialogView = LayoutInflater.from(this).inflate(R.layout.dialog_task, null);
            EditText taskText = dialogView.findViewById(R.id.task);
            new AlertDialog.Builder(ProjectsActivity.this)
                    .setTitle("Add a new project")
                    .setView(dialogView)
                    .setPositiveButton("Add", (dialog, which) -> realm.executeTransactionAsync(realm -> {
                        Project project = new Project();
                        String userId = SyncUser.currentUser().getIdentity();
                        String name = taskText.getText().toString();

                        project.setId(UUID.randomUUID().toString());
                        project.setOwner(userId);
                        project.setName(name);
                        project.setTimestamp(new Date());

                        realm.insert(project);
                    }))
                    .setNegativeButton("Cancel", null)
                    .create()
                    .show();
        });

        // using the current SyncUser#id, perform a partial query to obtain
        // only projects belonging to this SyncUser.
        realm = Realm.getDefaultInstance();
        RealmResults<Project> projects = realm
                .where(Project.class)
                .equalTo("owner", SyncUser.currentUser().getIdentity())
                .sort("timestamp", Sort.DESCENDING)
                .findAllAsync();

        final ProjectsRecyclerAdapter itemsRecyclerAdapter = new ProjectsRecyclerAdapter(this, projects);
        RecyclerView recyclerView = findViewById(R.id.recycler_view);
        recyclerView.setLayoutManager(new LinearLayoutManager(this));
        recyclerView.setAdapter(itemsRecyclerAdapter);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        realm.close();
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.menu_items, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        if (item.getItemId() == R.id.action_logout) {
            SyncUser syncUser = SyncUser.currentUser();
            if (syncUser != null) {
                syncUser.logout();
                Intent intent = new Intent(this, WelcomeActivity.class);
                intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK);
                startActivity(intent);
            }
            return true;
        }
        return super.onOptionsItemSelected(item);
    }
}
```

You will also have to add this new activity to the manifest:

```markup
<activity
            android:name=".ProjectsActivity"
            android:label="@string/title_select_projects"
            android:theme="@style/AppTheme.NoActionBar" />
```

And this in turn requires that you add the following to `strings.xml`:

```text
<string name="title_select_projects">My Projects</string>
```

### Add the RecyclerView Adapter {#add-the-recyclerview-adapter}

Add a new class `ProjectsRecyclerAdapter` under the `ui` package, this the the adaptor of the recycler view, used by the new `ProjectActivity`

* Note that for simplicity we attach an `onClickListener` when binding the `ViewHolder` . This will navigate to the `ItemsAcivity` by passing in argument the `project_id` This way the `ItemsAcivity` knows which list of `Item` it needs to display

```java
holder.textView.setOnClickListener(v -> {
    Intent intent = new Intent(context, ItemsActivity.class);
    intent.putExtra("project_id", project.getId());
    context.startActivity(intent);
});
```

The rest of the logic is similar to the `ItemsRecyclerAdapter`

```java
package io.realm.todo.ui;

import android.content.Context;
import android.content.Intent;
import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import io.realm.OrderedRealmCollection;
import io.realm.RealmRecyclerViewAdapter;
import io.realm.todo.ItemsActivity;
import io.realm.todo.model.Project;

public class ProjectsRecyclerAdapter extends RealmRecyclerViewAdapter<Project, ProjectsRecyclerAdapter.MyViewHolder> {
    private final Context context;

    public ProjectsRecyclerAdapter(Context context, OrderedRealmCollection<Project> data) {
        super(data, true);
        this.context = context;
    }

    @Override
    public MyViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View itemView = LayoutInflater.from(parent.getContext())
                .inflate(android.R.layout.simple_list_item_1, parent, false);
        return new MyViewHolder(itemView);
    }

    @Override
    public void onBindViewHolder(MyViewHolder holder, int position)  {
        final Project project = getItem(position);
        if (project != null) {
            holder.textView.setText(project.getName());
            holder.textView.setOnClickListener(v -> {
                Intent intent = new Intent(context, ItemsActivity.class);
                intent.putExtra("project_id", project.getId());
                context.startActivity(intent);
            });
        }
    }

    class MyViewHolder extends RecyclerView.ViewHolder {
        TextView textView;

        MyViewHolder(View itemView) {
            super(itemView);
            textView = itemView.findViewById(android.R.id.text1);
        }
    }
}
```

## Tweaking ItemsActivity {#tweaking-itemsactivity}

Since we're getting to this Activity from `ProjectsActivity` we need to obtain the id of the project we want to display, this is done inside `onCreate`

```java
String projectId = getIntent().getStringExtra("project_id");
```

This will allow us to query for the corresponding project then display it's tasks

```java
mRealm = Realm.getDefaultInstance();
Project project = realm.where(Project.class).equalTo("id", projectId).findFirst();
```

```java
ItemsRecyclerAdapter itemsRecyclerAdapter = new ItemsRecyclerAdapter(
project.getTasks().sort("timestamp", Sort.ASCENDING));
...
```

Complete version

```java
package io.realm.todo;

import android.content.Intent;
import android.os.Bundle;
import android.support.v7.app.AlertDialog;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.support.v7.widget.helper.ItemTouchHelper;
import android.view.LayoutInflater;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.widget.EditText;

import io.realm.Realm;
import io.realm.Sort;
import io.realm.SyncUser;
import io.realm.todo.model.Item;
import io.realm.todo.model.Project;
import io.realm.todo.ui.ItemsRecyclerAdapter;

public class ItemsActivity extends AppCompatActivity {

    private Realm realm;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_items);

        setSupportActionBar(findViewById(R.id.toolbar));

        String projectId = getIntent().getStringExtra("project_id");

        findViewById(R.id.fab).setOnClickListener(view -> {
            View dialogView = LayoutInflater.from(this).inflate(R.layout.dialog_task, null);
            EditText taskText = dialogView.findViewById(R.id.task);
            new AlertDialog.Builder(ItemsActivity.this)
                    .setTitle("Add a new task")
                    .setMessage("What do you want to do next?")
                    .setView(dialogView)
                    .setPositiveButton("Add", (dialog, which) -> realm.executeTransactionAsync(realm -> {
                        Item item = new Item();
                        item.setBody(taskText.getText().toString());
                        realm.where(Project.class).equalTo("id", projectId).findFirst().getTasks().add(item);
                    }))
                    .setNegativeButton("Cancel", null)
                    .create()
                    .show();
        });

        realm = Realm.getDefaultInstance();
        Project project = realm.where(Project.class).equalTo("id", projectId).findFirst();

        setTitle(project.getName());
        final ItemsRecyclerAdapter itemsRecyclerAdapter = new ItemsRecyclerAdapter(project.getTasks().sort("timestamp", Sort.ASCENDING));
        RecyclerView recyclerView = findViewById(R.id.recycler_view);
        recyclerView.setLayoutManager(new LinearLayoutManager(this));
        recyclerView.setAdapter(itemsRecyclerAdapter);

        ItemTouchHelper.SimpleCallback simpleItemTouchCallback = new ItemTouchHelper.SimpleCallback(0, ItemTouchHelper.LEFT | ItemTouchHelper.RIGHT) {

            @Override
            public boolean onMove(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder, RecyclerView.ViewHolder target) {
                return false;
            }

            @Override
            public void onSwiped(RecyclerView.ViewHolder viewHolder, int swipeDir) {
                int position = viewHolder.getAdapterPosition();
                String id = itemsRecyclerAdapter.getItem(position).getItemId();
                realm.executeTransactionAsync(realm -> {
                    Item item = realm.where(Item.class).equalTo("itemId", id)
                            .findFirst();
                    if (item != null) {
                        item.deleteFromRealm();
                    }
                });
            }
        };

        ItemTouchHelper itemTouchHelper = new ItemTouchHelper(simpleItemTouchCallback);
        itemTouchHelper.attachToRecyclerView(recyclerView);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        realm.close();
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.menu_items, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        if (item.getItemId() == R.id.action_logout) {
            SyncUser syncUser = SyncUser.currentUser();
            if (syncUser != null) {
                syncUser.logout();
                Intent intent = new Intent(this, WelcomeActivity.class);
                intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK);
                startActivity(intent);
            }
            return true;
        }
        return super.onOptionsItemSelected(item);
    }
}
```

## Compile and Run! {#compile-and-run!}

Now you can use two different users on two different devices, you should only see each user's project/tasks.

{% hint style="info" %}
You can also pull the Realm file from the device then open it using [Realm Studio](../../realm-studio/), you'll see effectively that your local Realm contains only `Item` and `Project` related to your user.
{% endhint %}

![](https://blobscdn.gitbook.com/v0/b/gitbook-28427.appspot.com/o/assets%2F-L22PAstP49i9mPl4sPN%2F-L3ie62rQOBxzmQJGkjW%2F-L3ieAaekz-HwrwKeBkp%2FPartialSync.gif?alt=media&token=a60749b3-b0c0-4d1d-b787-cee0cf98ad90)

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3)

