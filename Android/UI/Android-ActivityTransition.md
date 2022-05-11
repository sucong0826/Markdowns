## Activity Transition in Android (1st Chapter)

### Before Writing
----------

It is the first time that I've written a blog in English. I try to write blogs in English, Whatever I am little excited now.

The blog is about Activity Transition in Android. Activity Transition is a branch of Transition. The reason why I choose this function is because I practice samples that Google provides recently. And the first one is Activity Transition. Now, Let's start.

### Category
1. What is Activity Transition ?
2. How to use it ?
3. The principle of Activity Transition.
4. A sample.


### 1.What is Activity Transition?
------
Activity Transition is just a scene where plays an animation via intent when an activity starts another, that is, **it is a kind of animation.**

Let's see how it works by displaying a GIF.

<img src="/ActivityTransition.gif" alt="" width="30%" height="30%" />

The sample is provided by Google from Android samples. When we click one of those items, the item will start another Activity. In the progress of starting, it looks like that the picture on it is expanded and then filled the width of the screen in the started Activity, it does actually. Well I'm really curious about this, so I gonna figure it out to get the principle about it.

Let's see the code about when I click the item.

```Java
  public void clicked(View view) {
      mHero = (ImageView) view;
      Intent intent = new Intent(this, ActivityTransitionDetails.class);
      intent.putExtra(KEY_ID, ViewCompat.getTransitionName(mHero));
      ActivityOptionsCompat activityOptionsCompat = ActivityOptionsCompat.makeSceneTransitionAnimation(this, mHero, "hero");
      ActivityCompat.startActivity(this, intent, activityOptionsCompat.toBundle());
  }
```
This snippet of code does start an activity as we see. First of all, it creates an intent with arguments `this` and `ActivityTransitionDetails.class`. Then the `intent` puts an string extra, the name is `KEY_ID`, the value is `ViewCompat.getTransitionName(mHero)`. Actually `KEY_ID` has a string value of `"ViewTransitionValues:id"`. It is easy to be understood. However, what is `ViewCompat.getTransitionName(mHero);`? Alright let's search it in official APIs!

```Java
    /**
     * Returns the name of the View to be used to identify Views in Transitions.
     * Names should be unique in the View hierarchy.
     *
     * <p>This returns null if the View has not been given a name.</p>
     *
     * @param view The View against which to invoke the method.
     * @return The name used of the View to be used to identify Views in Transitions or null
     * if no name has been given.
     */
    public static String getTransitionName(View view) {
        return IMPL.getTransitionName(view);
    }
```

`ViewCompat` is added in version 22.0.0. It is a helper class for accessing features in `View` introduced after API level 4 backwards compatible fashion. Therefore the method `ViewCompat.getTransitionName(View view)` is for accessing features of View. Document of the method says that "*Returns the name of the View to be used to identify Views in Transitions*". There are three words as keys. The first is "**name**". The second is "**identify**". The third is "**Transitions**".

Ok, let's talk about these three key words.

What name? Please see the document of method `getTransitionName(View view)`, in declaration, "*The name used of the View to be used to identify Views in Transitions or null if no name has been given.*" and "*Names should be unique in the View hierarchy.*". The name should be unique. The name identifies a View in Transitions. A Transition holds information about animations that will be run on its target during a scene change as we mentioned above. `IMPL.getTransitionName(view)` locates the call in code of `View`.

```Java
    /**
     * Returns the name of the View to be used to identify Views in Transitions.
     * Names should be unique in the View hierarchy.
     *
     * <p>This returns null if the View has not been given a name.</p>
     *
     * @return The name used of the View to be used to identify Views in Transitions or null
     * if no name has been given.
     */
    @ViewDebug.ExportedProperty
    public String getTransitionName() {
        return mTransitionName;
    }
```

This method just return a value named `mTransitionName`. Thus I can infer that `mTransitionName` is a property of View. It presents in layout XML as expected.

```XML
    <ImageView
        android:id="@+id/ducky"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_column="0"
        android:layout_row="0"
        android:onClick="clicked"
        android:scaleType="centerCrop"
        android:src="@drawable/ducky"
        android:transitionName="ducky" />
```

```XML
android:transitionName="ducky"
```
the value of ***android:transitionName*** is just the name we find. As above, the name is unique and it identifies a View in Transitions.  Therefore in the gif, there are six ImageViews and each one has a unique ***android:transitionName*** defined in layout XML.

Well, please turn back to the code about staring another activity. The first line in `clicked(View view)` method is a casting. The view as passed-in argument is casted to an ImageView which has a property named ***android:transitionName***. The second line is about initiating an intent to be started. This line is easy. The third line is putting a string extra to the intent and we cleared the source and meaning of `ViewCompat.getTransitionName(View view)`. Next line is important and I am going to illustrate much more on it.

`ActivityOptionsCompat activityOptionsCompat = ActivityOptionsCompat.makeSceneTransitionAnimation(this, mHero, "hero");`

Let me introduce the class `ActivityOptionsCompat`. It is the 'compat' version of `android.app.ActivityOptions`. The functionality of `ActivityOptions` is same as `ViewCompat` but it severs `Activity`.

> *Helper class for building an options Bundle that can be used with Context.startActivity(Intent, Bundle) and related methods.*

A method named `ActivityOptionsCompat.makeSceneTransitionAnimation(this, mHero, "hero")` works. Well let's go inside to explore. In `ActivityOptions`.

So many terms are needed to be explained. As above, the method `ActivityOptionsCompat.makeSceneTransitionAnimation(this, mHero, "hero")` has two key arguments, `mHero` and `hero`. `mHero` is the ImageView that we clicked and `hero` is a shared element name. `hero` is always a string constant that you define it as you want.

`ActivityOptions` provides the basic method `makeSceneTransitionAnimation(Activity activity, View sharedElement, String sharedElementName);`. Document of it says:

> *Create an ActivityOptions to transition between Activities using cross-Activity scene animations. This method carries the position of one shared element to the started Activity. The position of <code>sharedElement</code> will be used as the epicenter for the exit Transition. The position of the shared element in the launched Activity will be the epicenter of its entering Transition.*

Note that "*This method carries the position of one shared element to the started Activity.*". "*one shared element*" the words speak of is the view that we click. **Each view on screen will be clicked is one shared element**. There is a start animation and an exit animation in gif above. Therefore the position of shared element will be used as the center for the exit Transition and the position of the shared element in the launched Activity will be the epicenter of its entering Transition.

Let's explain it.
![Explain Transition](/Explain Transition.jpg)

Look at the picture, it explains how a Transition works. If a shared element (view) wants to recover to its previous position after another Activity being started, the view's position must be saved by some way.

The chain of invoking from outside to inside:

1. In *ActivityTransition.java*:
```Java
ActivityOptionsCompat activityOptionsCompat = ActivityOptionsCompat.makeSceneTransitionAnimation(this, mHero, "hero");
```

2. In *ActivityOptions.java*:
```Java
public static ActivityOptions makeSceneTransitionAnimation(Activity activity, View sharedElement, String sharedElementName);
```

3. In *ActivityOptions.java*: in this step, View of shared element and string of shared element name is encapsulated into a Pair&lt;View, String&gt; array. Then call a method named `makeSceneTransitionAnimation(Activity, Window, ActivityOptions, Listener, Pair...)`
```Java
    @SafeVarargs
    public static ActivityOptions makeSceneTransitionAnimation(Activity activity,
            Pair<View, String>... sharedElements) {
        ActivityOptions opts = new ActivityOptions();
        makeSceneTransitionAnimation(activity, activity.getWindow(), opts,
                activity.mExitTransitionListener, sharedElements);
        return opts;
    }
```

4. In *ActivityOptions.java*: First of all, Window must have the feature of `Window.FEATURE_ACTIVITY_TRANSITIONS`, return `null` otherwise. And then the for-loop is aim at parsing `Pair<View, String>`s to two lists, the one contains shared elements (views), the other contains shared element names, both cannot be null. In the end, there creates an instance of class `ExitTransitionCoordinator` with the parameters given above. Regardless of the principle, we know that the method returns an instance of *ExitTransitionCoordinator*. Let's peek the class document for more details. *This ActivityTransitionCoordinator is created in ActivityOptions#makeSceneTransitionAnimation to govern the exit of the Scene and the shared elements when calling an Activity as well as the reentry of the Scene when coming back from the called Activity.* The result of reading this quote, no matter exit or reentry, `ActivityTransitionCoordinator` is going to take in charge.
```Java
static ExitTransitionCoordinator makeSceneTransitionAnimation(Activity activity, Window window,
            ActivityOptions opts, SharedElementCallback callback,
            Pair<View, String>[] sharedElements) {
        if (!window.hasFeature(Window.FEATURE_ACTIVITY_TRANSITIONS)) {
            opts.mAnimationType = ANIM_DEFAULT;
            return null;
        }
        opts.mAnimationType = ANIM_SCENE_TRANSITION;

        ArrayList<String> names = new ArrayList<String>();
        ArrayList<View> views = new ArrayList<View>();

        if (sharedElements != null) {
            for (int i = 0; i < sharedElements.length; i++) {
                Pair<View, String> sharedElement = sharedElements[i];
                String sharedElementName = sharedElement.second;
                if (sharedElementName == null) {
                    throw new IllegalArgumentException("Shared element name must not be null");
                }
                names.add(sharedElementName);
                View view = sharedElement.first;
                if (view == null) {
                    throw new IllegalArgumentException("Shared element must not be null");
                }
                views.add(sharedElement.first);
            }
        }

        ExitTransitionCoordinator exit = new ExitTransitionCoordinator(activity, window,
                callback, names, names, views, false);
        ...
        return exit;
    }
```

5. In *ActivityOptions.java*, there goes a method to start the transition tought it is added by @Hide annotation. Please note this line, `exit.startExit();`, method `startExit()` in `ExitTransitionCoordinator` will be called.
```Java
    @SafeVarargs
    public static ActivityOptions startSharedElementAnimation(Window window,
            Pair<View, String>... sharedElements) {
        ActivityOptions opts = new ActivityOptions();
        final View decorView = window.getDecorView();
        if (decorView == null) {
            return opts;
        }
        final ExitTransitionCoordinator exit =
                makeSceneTransitionAnimation(null, window, opts, null, sharedElements);
        if (exit != null) {
            HideWindowListener listener = new HideWindowListener(window, exit);
            exit.setHideSharedElementsCallback(listener);
            exit.startExit();
        }
        return opts;
    }
```

6. In *ExitTransitionCoordinator.java*, method `void startExit()` will be called.
```Java
    public void startExit() {
        if (!mIsExitStarted) {
            ...
            startTransition(new Runnable() {
                @Override
                public void run() {
                    if (mActivity != null) {
                        beginTransitions();
                    } else {
                        startExitTransition();
                    }
                }
            });
        }
    }
```

7. In *ExitTransitionCoordinator.java*, a private method named `void beginTransitions()` will be called to begin the transition.
```Java
    private void beginTransitions() {
        Transition sharedElementTransition = getSharedElementExitTransition();
        Transition viewsTransition = getExitTransition();

        Transition transition = mergeTransitions(sharedElementTransition, viewsTransition);
        ViewGroup decorView = getDecor();
        if (transition != null && decorView != null) {
            ...
            TransitionManager.beginDelayedTransition(decorView, transition);
            ...
            decorView.invalidate();
        } else {
            transitionStarted();
        }
    }
```

8. *ActivityTransition.java*: Just start another Activity. In this line of code, activityOptionsCompat needs to call `toBundle()` method, some values and key properties should be put into a Bundle delivered to the started Activity.
```Java
    ActivityCompat.startActivity(this, intent, activityOptionsCompat.toBundle());

    // toBundle();
    if (mTransitionReceiver != null) {
        b.putParcelable(KEY_TRANSITION_COMPLETE_LISTENER, mTransitionReceiver);
    }
    b.putBoolean(KEY_TRANSITION_IS_RETURNING, mIsReturning);
    b.putStringArrayList(KEY_TRANSITION_SHARED_ELEMENTS, mSharedElementNames);
    b.putParcelable(KEY_RESULT_DATA, mResultData);
    b.putInt(KEY_RESULT_CODE, mResultCode);
    b.putInt(KEY_EXIT_COORDINATOR_INDEX, mExitCoordinatorIndex);
```

### 2.How To Use It? ###
------
Waiting...

### 3. The principle of Activity Transition. ###
------
This part is hard a little bit. I'm studying...

### 4. A Sample
------
I will write a good sample when I get everything about it totally.
