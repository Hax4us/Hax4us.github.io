---
layout: post
title: Custom Layout In Navigation View
subtitle: Yes it's possible customize naigation view in drawer layout
tags: [android, java]
---
If you are like me who just finished his mountain due and now want to do something **toofani** then you are at right place. 

Let's make a navigation view without menu items and inject our own layout with our own menu items ( however they are not menu items exactly )

Let's short list the steps, we are going to follow and complete in sequence.

1. Make a custom layout for navigation view
    * Custom Header for navigation view.
    * Custom Menu for navigation view (via ListView).
        * A separate layout for item of ListView.

2. Make a custom Drawer Item which behaves like a menu item
    * Pojo for our items.
    * Custom Adapter for ListView.

3. Bind everything together in our Activity.

### Custom Layout For Navigation View
---
* Current layout of our Activity (which has navigation view)
    
    `activity_main.xml`

    ```xml
    <androidx.drawerlayout.widget.DrawerLayout
        xmlns:android="http://schemas.android.com/apk/res"
        android:id="drawer_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <include layout="@layout/main_content" />

        <com.google.android.material.navigation.NavigationView
            android:layout_width="wrap_content"
            android:layout_height="match_parent"
            android:layout_gravity="start">

            <!-- Here we will place our custom layout -->

        </com.google.android.material.navigation.NavigationView>
    </androidx.drawerlayout.widget.DrawerLayout>
    ```


* Ok so first make a separate layout for our navigation view (i like to make layout's in separate xml files and then include by `<include />`)
    
    `custom_navigation_layout.xml`

    ```xml
    <LinearLayout
        xmlns:android="http://schemas.android.com/apk/res"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <include layout="@layout/nav_header"
        
        <include layout="@layout/nav_body"

    </LinearLayout>
    ```

    `nav_header.xml`

    ```xml
    <FrameLayout
        xmlns:android="http://schemas.android.com/apk/res"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_gravity="center" 
            android:text="Nav Header"/>

    </FrameLayout>
    ```

    `nav_body.xml` (where your actions will be placed)

    ```xml
    <FrameLayout
        xmlns:android="http://schemas.android.com/apk/res"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <ListView
            android:id="@+id/nav_list_view"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />

    </FrameLayout>
    ```


* Now `activity_main.xml` becomes 

    ```xml
    <androidx.drawerlayout.widget.DrawerLayout
        xmlns:android="http://schemas.android.com/apk/res"
        android:id="@+id/drawer_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <include layout="@layout/main_content" />

        <com.google.android.material.navigation.NavigationView
            android:layout_width="wrap_content"
            android:layout_height="match_parent"
            android:layout_gravity="start">

            <!-- Here we will place our custom layout -->
            <include layout="@layout/custom_navigation_layout />"

        </com.google.android.material.navigation.NavigationView>
    </androidx.drawerlayout.widget.DrawerLayout>
    ```


* One last thing left is layout for our items of `ListView`. Ok so let's make it then 

    `item_nav_list_view.xml`

    ```xml
    <LinearLayout 
        xmlns:android="http://schemas.android.com/apk/res"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <ImageView
            android:id="@+id/item_icon"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />

        <TextView
            android:id="@+id/item_name"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />

    </LinearLayout>
    ```

* Well so now everything is done by layout side. Let's move on the code (java) part.


### Custom Drawer Item
---

* Pojo For Drawer Item
    
    `DrawerItem.java`

    ```java
    public class DrawerItem {
        private String itemName;
        @DrawableRes
        private int itemIcon;

        public DrawerItem(String itemName, @DrawableRes int itemIcon) {
            this.itemName = itemName;
            this.itemIcon = itemIcon;
        }

        public String getItemName() {
            return itemName;
        }

        public String getItemIcon() {
            return itemIcon;
        }
    }
    ```


* Our custom ArrayAdapter

    `NavArrayAdapter.java`

    ```java
    public class NavArrayAdapter extends ArrayAdapter<DrawerItem> {
        private Context mContext;
        @LayoutRes
        private int mLayoutResId;
        private List<DrawerItem> mDrawerItems;

        public NavArrayAdapter(Context context,
                @LayoutRes int layoutResId,
                List<DrawerItem> drawerItems) {
            mContext = context;
            mLayoutResId = layoutResId;
            mDrawerItems = drawerItems;

            super(mContext, mLayoutResId, mDrawerItems)
        }

        @Override
        public View getView() {
            if (convertView == null) {
                convertView =
                    LayoutInflater.from(mContext)
                        .inflate(mLayoutResId, 
                                parent,
                                false);
            }

            ImageView icon = convertView.findViewById(R.id.item_icon);
            TextView name = convertView.findViewById(R.id.item_name);

            icon.setImageResource(drawerItems.get(position).getItemIcon());
            name.setText(drawerItems.get(position).getItemName());

            return convertView; 
        }
    }
    ```


### Bind everything together
---

* `MainActivity.java`

    ```java
    public class MainActivity extends AppCompatActivity {

        private DrawerLayout drawerLayout;
        private ListView navActionItems;
        private NavArrayAdapter navArrayAdapter;

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);

            drawerLayout = findViewById(R.id.drawer_layout);
            navActionItems = findViewById(R.id.nav_list_view);

            List<DrawerItem> items = new ArrayList<>();
            items.add(new DrawerItem("Item 1", R.drawable.ic_item_1));
            items.add(new DrawerItem("Item 2", R.drawable.ic_item_2));
            items.add(new DrawerItem("Item 3", R.drawable.ic_item_3));

            navArrayAdapter = new NavArrayAdapter(this, 
                    R.layout.item_nav_list_view,
                    items);

            navActionItems.setAdapter(navArrayAdapter);

        }
    }
    ```

Well it was my approach and if you are using any different approach so let me know in comments :)

