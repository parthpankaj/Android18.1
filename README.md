# Android18.1
package com.example.pankaj.googleanalytics;

import android.app.Application;

import com.google.android.gms.analytics.GoogleAnalytics;
import com.google.android.gms.analytics.HitBuilders;
import com.google.android.gms.analytics.StandardExceptionParser;
import com.google.android.gms.analytics.Tracker;

/**
 * Created by Ravi on 13/08/15.
 */
public class MyApplication extends Application {
    public static final String TAG = MyApplication.class
            .getSimpleName();

    private static MyApplication mInstance;

    @Override
    public void onCreate() {
        super.onCreate();
        mInstance = this;

        AnalyticsTrackers.initialize(this);
        AnalyticsTrackers.getInstance().get(AnalyticsTrackers.Target.APP);
    }

    public static synchronized MyApplication getInstance() {
        return mInstance;
    }

    public synchronized Tracker getGoogleAnalyticsTracker() {
        AnalyticsTrackers analyticsTrackers = AnalyticsTrackers.getInstance();
        return analyticsTrackers.get(AnalyticsTrackers.Target.APP);
    }

    /***
     * Tracking screen view
     *
     * @param screenName screen name to be displayed on GA dashboard
     */
    public void trackScreenView(String screenName) {
        Tracker t = getGoogleAnalyticsTracker();

        // Set screen name.
        t.setScreenName(screenName);

        // Send a screen view.
        t.send(new HitBuilders.ScreenViewBuilder().build());

        GoogleAnalytics.getInstance(this).dispatchLocalHits();
    }

    /***
     * Tracking exception
     *
     * @param e exception to be tracked
     */
    public void trackException(Exception e) {
        if (e != null) {
            Tracker t = getGoogleAnalyticsTracker();

            t.send(new HitBuilders.ExceptionBuilder()
                    .setDescription(
                            new StandardExceptionParser(this, null)
                                    .getDescription(Thread.currentThread().getName(), e))
                    .setFatal(false)
                    .build()
            );
        }
    }

    /***
     * Tracking event
     *
     * @param category event category
     * @param action   action of the event
     * @param label    label
     */
    public void trackEvent(String category, String action, String label) {
        Tracker t = getGoogleAnalyticsTracker();

        // Build and send an Event.
        t.send(new HitBuilders.EventBuilder().setCategory(category).setAction(action).setLabel(label).build());
    }

}
---------------------------
package com.example.pankaj.googleanalytics;


import android.content.Context;

import com.google.android.gms.analytics.GoogleAnalytics;
import com.google.android.gms.analytics.Tracker;

import java.util.HashMap;
import java.util.Map;

/**
 * A collection of Google Analytics trackers. Fetch the tracker you need using
 * {@code AnalyticsTrackers.getInstance().get(...)}
 * <p/>
 * This code was generated by Android Studio but can be safely modified by
 * hand at this point.
 * <p/>
 * TODO: Call {@link #initialize(Context)} from an entry point in your app
 * before using this!
 */
public final class AnalyticsTrackers {

    public enum Target {
        APP,
        // Add more trackers here if you need, and update the code in #get(Target) below
    }

    private static AnalyticsTrackers sInstance;

    public static synchronized void initialize(Context context) {
        if (sInstance != null) {
            throw new IllegalStateException("Extra call to initialize analytics trackers");
        }

        sInstance = new AnalyticsTrackers(context);
    }

    public static synchronized AnalyticsTrackers getInstance() {
        if (sInstance == null) {
            throw new IllegalStateException("Call initialize() before getInstance()");
        }

        return sInstance;
    }

    private final Map<Target, Tracker> mTrackers = new HashMap<Target, Tracker>();
    private final Context mContext;

    /**
     * Don't instantiate directly - use {@link #getInstance()} instead.
     */
    private AnalyticsTrackers(Context context) {
        mContext = context.getApplicationContext();
    }

    public synchronized Tracker get(Target target) {
        if (!mTrackers.containsKey(target)) {
            Tracker tracker;
            switch (target) {
                case APP:
                    tracker = GoogleAnalytics.getInstance(mContext).newTracker(R.xml.app_tracker);
                    break;
                default:
                    throw new IllegalArgumentException("Unhandled analytics target " + target);
            }
            mTrackers.put(target, tracker);
        }

        return mTrackers.get(target);
    }
}
--------------
package com.example.pankaj.googleanalytics;

import android.content.Intent;
import android.os.Bundle;
import android.os.Handler;
import android.support.v4.app.FragmentManager;
import android.support.v4.app.FragmentTransaction;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.Toolbar;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {

    private static String TAG = MainActivity.class.getSimpleName();

    private Toolbar mToolbar;

    private Button btnSecondScreen, btnSendEvent, btnException, btnAppCrash, btnLoadFragment;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mToolbar = (Toolbar) findViewById(R.id.toolbar);
        btnSecondScreen = (Button) findViewById(R.id.btnSecondScreen);
        btnSendEvent = (Button) findViewById(R.id.btnSendEvent);
        btnException = (Button) findViewById(R.id.btnException);
        btnAppCrash = (Button) findViewById(R.id.btnAppCrash);
        btnLoadFragment = (Button) findViewById(R.id.btnLoadFragment);

        setSupportActionBar(mToolbar);
        getSupportActionBar().setDisplayShowHomeEnabled(true);

        /**
         * Launching another activity to track the other screen
         */
        btnSecondScreen.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Intent intent = new Intent(MainActivity.this, SecondActivity.class);
                startActivity(intent);
            }
        });
    }
}
---------------------------
package com.example.pankaj.googleanalytics;

import android.app.Activity;
import android.os.Bundle;

/**
 * Created by Pankaj on 19-Dec-16.
 */
public class SecondActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
}
-------
<?xml version="1.0" encoding="utf-8"?>
<android.support.v7.widget.Toolbar xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:local="http://schemas.android.com/apk/res-auto"
    android:id="@+id/toolbar"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:minHeight="?attr/actionBarSize"
    android:background="?attr/colorPrimary"
    android:elevation="2dp"
    local:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
    local:popupTheme="@style/ThemeOverlay.AppCompat.Light" />
    --------------------------------------------
    <?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:clipToPadding="false">

    <include
        android:id="@+id/toolbar"
        layout="@layout/toolbar"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentTop="true"/>

    <LinearLayout
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:layout_below="@id/toolbar"
        android:orientation="vertical">

        <TextView
            android:layout_width="fill_parent"
            android:gravity="center_horizontal"
            android:padding="10dp"
            android:layout_height="wrap_content"
            android:text="@string/msg_instructions" />

        <Button android:id="@+id/btnSecondScreen"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:background="@color/btnBackground"
            android:text="@string/btn_second_screen"
            android:textAllCaps="false"
            android:paddingLeft="15dp"
            android:paddingRight="15dp"
            android:textColor="#797979"
            android:layout_gravity="center_horizontal"
            android:elevation="2dp"
            android:layout_marginTop="20dp"/>

        <Button android:id="@+id/btnSendEvent"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:background="@color/btnBackground"
            android:text="@string/btn_send_event"
            android:textAllCaps="false"
            android:paddingLeft="15dp"
            android:paddingRight="15dp"
            android:textColor="#797979"
            android:layout_gravity="center_horizontal"
            android:elevation="2dp"
            android:layout_marginTop="20dp"/>

        <Button android:id="@+id/btnException"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:background="@color/btnBackground"
            android:text="@string/btn_exception_tracking"
            android:textAllCaps="false"
            android:paddingLeft="15dp"
            android:paddingRight="15dp"
            android:textColor="#797979"
            android:layout_gravity="center_horizontal"
            android:elevation="2dp"
            android:layout_marginTop="20dp"/>

        <Button android:id="@+id/btnAppCrash"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:background="@color/btnBackground"
            android:text="@string/btn_app_crash"
            android:textAllCaps="false"
            android:paddingLeft="15dp"
            android:paddingRight="15dp"
            android:textColor="#797979"
            android:layout_gravity="center_horizontal"
            android:elevation="2dp"
            android:layout_marginTop="20dp"/>

        <Button android:id="@+id/btnLoadFragment"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:background="@color/btnBackground"
            android:text="@string/btn_load_fragment"
            android:textAllCaps="false"
            android:paddingLeft="15dp"
            android:paddingRight="15dp"
            android:textColor="#797979"
            android:layout_gravity="center_horizontal"
            android:elevation="2dp"
            android:layout_marginTop="20dp"/>

    </LinearLayout>

    <FrameLayout
        android:id="@+id/frame_container"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true" />
</RelativeLayout>
-------------------------------------------------------------
<?xml version="1.0" encoding="utf-8"?>
<resources>s>
        <color name="colorPrimary">#FF6D00</color>
        <color name="colorPrimaryDark">#FF6D00</color>
        <color name="textColorPrimary">#FFFFFF</color>
        <color name="windowBackground">#FFFFFF</color>
        <color name="navigationBarColor">#000000</color>
        <color name="colorAccent">#FFD180</color>
        <color name="btnBackground">#e7e7e7</color>

</resources>
