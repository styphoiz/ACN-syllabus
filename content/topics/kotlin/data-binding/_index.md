---
_db_id: 390
content_type: topic
prerequisites:
  hard:
  - topics/kotlin/in-line-functions
  soft: []
ready: true
title: Data Binding
---

First of all, after having a created Android project in Android Studio, we need to add the Data Binding dependency and the ones of Kotlin to the build.gradle file of our app.
```
apply plugin: 'kotlin-android'                       
apply plugin: 'kotlin-kapt'
android {
    ....
    dataBinding {
        enabled = true
    }
}
dependencies {
    ...
    // notice that the compiler version must be the same than our gradle version
    kapt 'com.android.databinding:compiler:2.3.1'
}
```
That’s all the configuration we need to start using Data Binding with Kotlin. Thank you very much for reading me… Now lets continue with the fun starting to see the code.

First we need to create a model. In this case a basic one like User.kt
data class User(val name: String, val age: Int)

In our activity_main.xml we can do something like this:
```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
    >
<!-- Inside the layout tag it is possible to set the data tag in order to set one or many variables. For this example we are having an User property-->
    <data>

        <variable
            name="user"
            type="com.kuma.sample.User"
            />
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        tools:context="com.kuma.sample.MainActivity"
        >

        <TextView
            android:id="@+id/user_name_text_view"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:padding="16dp"
            android:text="@{user.name}"
            tools:text="Name"
            />

        <TextView
            android:id="@+id/user_age_text_view"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:padding="16dp"
            android:text="@{Integer.toString(user.age)}"
            tools:text="XX"
            />

    </LinearLayout>

</layout>
```
Remember to always set your usual xml view inside the <layout> tag and attach all the “xmlns:” properties to it. Otherwise it will throw a compilation error, since the generated files will have duplicated properties.

### And here comes the Binding:
```
package com.kuma.sample

import android.databinding.DataBindingUtil
import android.os.Bundle
import android.support.v7.app.AppCompatActivity
import com.kuma.kotlinsteps.databinding.ActivityMainBinding

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val binding: ActivityMainBinding = DataBindingUtil.setContentView(this, R.layout.activity_main)

        val user = User("Kuma", 23)
        binding.setVariable(BR.user, user)
        binding.executePendingBindings()
    }

}
```
In that code snippet there are somethings to be noticed:
- Now exists a class called ActivityMainBinding, which is autogenerated from the activity_main.xml and this contains all the references to use the views that the xml contains.
- The way of making an instance of ActivityMainBinding is a little variation of the way of setting the xml layout to an activity.
- There is also a new BR class which is some kind of secondary R class used to store the variables declared on the data tag of the xml.
- After setting the variable to the binding object, it is necessary to call the executePendingBindings() in order to set the user variable attributes to the marked views.

After compiling this you’ll be able to see that the Data has been set to your view without the necessity of writing any 
```
textView.text = user.name
```
### Two-way data binding
Using one-way data binding, you can set a value on an attribute and set a listener that reacts to a change in that attribute:
```
<CheckBox
    android:id="@+id/rememberMeCheckBox"
    android:checked="@{viewmodel.rememberMe}"
    android:onCheckedChanged="@{viewmodel.rememberMeChanged}"
/>
```
Two-way data binding provides a shortcut to this process:
```
<CheckBox
    android:id="@+id/rememberMeCheckBox"
    android:checked="@={viewmodel.rememberMe}"
/>
```
The @={} notation, which importantly includes the "=" sign, receives data changes to the property and listen to user updates at the same time.

In order to react to changes in the backing data, you can make your layout variable an implementation of Observable, usually BaseObservable, and use a @Bindable annotation, as shown in the following code snippet:
```
class LoginViewModel : BaseObservable {
    // val data = ...

    @Bindable
    fun getRememberMe(): Boolean {
        return data.rememberMe
    }

    fun setRememberMe(value: Boolean) {
        // Avoids infinite loops.
        if (data.rememberMe != value) {
            data.rememberMe = value

            // React to the change.
            saveData()

            // Notify observers of a new value.
            notifyPropertyChanged(BR.remember_me)
        }
    }
}
```
Because the bindable property's getter method is called getRememberMe(), the property's corresponding setter method automatically uses the name setRememberMe().

### Two-way data binding using custom attributes

The platform provides two-way data binding implementations for the most common two-way attributes and change listeners, which you can use as part of your app. If you want to use two-way data binding with custom attributes, you need to work with the @InverseBindingAdapter and @InverseBindingMethod annotations.

For example, if you want to enable two-way data binding on a "time" attribute in a custom view called MyView, complete the following steps:

1.Annotate the method that sets the initial value and updates when the value changes using @BindingAdapter:

```
@BindingAdapter("time")
@JvmStatic fun setTime(view: MyView, newValue: Time) {
    // Important to break potential infinite loops.
    if (view.time != newValue) {
        view.time = newValue
    }
}
```
 2.Annotate the method that reads the value from the view using @InverseBindingAdapter:

```
@InverseBindingAdapter("time")
@JvmStatic fun getTime(view: MyView) : Time {
    return view.getTime()
}
```
At this point, data binding knows what to do when the data changes (it calls the method annotated with @BindingAdapter) and what to call when the view attribute changes (it calls the InverseBindingListener). However, it doesn't know when or how the attribute changes.

For that, you need to set a listener on the view. It can be a custom listener associated with your custom view, or it can be a generic event, such as a loss of focus or a text change. Add the @BindingAdapter annotation to the method that sets the listener for changes on the property:

```
@BindingAdapter("app:timeAttrChanged")
@JvmStatic fun setListeners(
        view: MyView,
        attrChange: InverseBindingListener
) {
    // Set a listener for click, focus, touch, etc.
}
```
The listener includes an InverseBindingListener as a parameter. You use the InverseBindingListener to tell the data binding system that the attribute has changed. The system can then start calling the method annotated using @InverseBindingAdapter, and so on.

Note: Every two-way binding generates a synthetic event attribute. This attribute has the same name as the base attribute, but it includes the suffix "AttrChanged". The synthetic event attribute allows the library to create a method annotated using @BindingAdapter to associate the event listener to the appropriate instance of View.
In practice, this listener includes some non-trivial logic, including listeners for one-way data binding. For an example, see the adapter for the text attribute change, TextViewBindingAdapter.

### Converters
If the variable that's bound to a View object needs to be formatted, translated, or changed somehow before being displayed, it's possible to use a Converter object.

For example, take an EditText object that shows a date:
```
<EditText
    android:id="@+id/birth_date"
    android:text="@={Converter.dateToString(viewmodel.birthDate)}"
/>
```
The viewmodel.birthDate attribute contains a value of type Long, so it needs to be formatted by using a converter.

Because a two-way expression is being used, there also needs to be an inverse converter to let the library know how to convert the user-provided string back to the backing data type, in this case Long. This process is done by adding the @InverseMethod annotation to one of the converters and have this annotation reference the inverse converter. An example of this configuration appears in the following code snippet:

```
object Converter {
    @InverseMethod("stringToDate")
    @JvmStatic fun dateToString(
        view: EditText, oldValue: Long,
        value: Long
    ): String {
        // Converts long to String.
    }

    @JvmStatic fun stringToDate(
        view: EditText, oldValue: String,
        value: String
    ): Long {
        // Converts String to long.
    }
}
```
### Infinite loops using two-way data binding
Be careful not to introduce infinite loops when using two-way data binding. When the user changes an attribute, the method annotated using @InverseBindingAdapter is called, and the value is assigned to the backing property. This, in turn, would call the method annotated using @BindingAdapter, which would trigger another call to the method annotated using @InverseBindingAdapter, and so on.

For this reason, it's important to break possible infinite loops by comparing new and old values in the methods annotated using @BindingAdapter.

### Two-way attributes
The platform provides built-in support for two-way data binding when you use the attributes in the following table. For details on how the platform provides this support, see the implementations for the corresponding binding adapters:

- https://developer.android.com/topic/libraries/data-binding/two-way

### Bind layout views to Architecture Components
The AndroidX library includes the __Architecture Components__, which you can use to design robust, testable, and maintainable apps. The Data Binding Library works seamlessly with the Architecture Components to further simplify the development of your UI. The layouts in your app can bind to the data in the Architecture Components, which already help you manage the UI controllers lifecycle and notify about changes in the data.

This page shows how to incorporate the Architecture Components to your app to further enhance the benefits of using the Data Binding Library.

### Use LiveData to notify the UI about data changes
You can use LiveData objects as the data binding source to automatically notify the UI about changes in the data. For more information about this Architecture Component, see LiveData Overview.

Unlike objects that implement Observable—such as observable fields—LiveData objects know about the lifecycle of the observers subscribed to the data changes. This knowledge enables many benefits, which are explained in The advantages of using LiveData. In Android Studio version 3.1 and higher, you can replace observable fields with LiveData objects in your data binding code.

To use a LiveData object with your binding class, you need to specify a lifecycle owner to define the scope of the LiveData object. The following example specifies the activity as the lifecycle owner after the binding class has been instantiated:

```
class ViewModelActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        // Inflate view and obtain an instance of the binding class.
        val binding: UserBinding = DataBindingUtil.setContentView(this, R.layout.user)

        // Specify the current activity as the lifecycle owner.
        binding.setLifecycleOwner(this)
    }
}

```
You can use a ViewModel component, as explained in Use ViewModel to manage UI-related data, to bind the data to the layout. In the ViewModel component, you can use the LiveData object to transform the data or merge multiple data sources. The following example shows how to transform the data in the ViewModel:


```
class ScheduleViewModel : ViewModel() {
    val userName: LiveData

    init {
        val result = Repository.userName
        userName = Transformations.map(result) { result -> result.value }
    }
}

```
### Use ViewModel to manage UI-related data
The Data Binding Library works seamlessly with ViewModel components, which expose the data that the layout observes and reacts to its changes. Using ViewModel components with the Data Binding Library allows you to move UI logic out of the layouts and into the components, which are easier to test. The Data Binding Library ensures that the views are bound and unbound from the data source when needed. Most of the remaining work consists in making sure that you're exposing the correct data. For more information about this Architecture Component, see ViewModel Overview.

To use the ViewModel component with the Data Binding Library, you must instantiate your component, which inherits from the ViewModel class, obtain an instance of your binding class, and assign your ViewModel component to a property in the binding class. The following example shows how to use the component with the library:


```
class ViewModelActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        // Obtain the ViewModel component.
        val userModel: UserModel by viewModels()

        // Inflate view and obtain an instance of the binding class.
        val binding: UserBinding = DataBindingUtil.setContentView(this, R.layout.user)

        // Assign the component to a property in the binding class.
        binding.viewmodel = userModel
    }
}

```
In your layout, assign the properties and methods of your ViewModel component to the corresponding views using binding expressions, as shown in the following example:

```
<CheckBox
    android:id="@+id/rememberMeCheckBox"
    android:checked="@{viewmodel.rememberMe}"
    android:onCheckedChanged="@{() -> viewmodel.rememberMeChanged()}" />

```
### Use an Observable ViewModel for more control over binding adapters
You can use a ViewModel component that implements the Observable to notify other app components about changes in the data, similar to how you would use a LiveData object.

There are situations where you might prefer to use a ViewModel component that implements the Observable interface over using LiveData objects, even if you lose the lifecycle management capabilities of LiveData. Using a ViewModel component that implements Observable gives you more control over the binding adapters in your app. For example, this pattern gives you more control over the notifications when data changes, it also allows you to specify a custom method to set the value of an attribute in two-way data binding.

To implement an observable ViewModel component, you must create a class that inherits from the ViewModel class and implements the Observable interface. You can provide your custom logic when an observer subscribes or unsubscribes to notifications using the addOnPropertyChangedCallback() and removeOnPropertyChangedCallback() methods. You can also provide custom logic that runs when properties change in the notifyPropertyChanged() method. The following code example shows how to implement an observable ViewModel:


```
/**
 * A ViewModel that is also an Observable,
 * to be used with the Data Binding Library.
 */
open class ObservableViewModel : ViewModel(), Observable {
    private val callbacks: PropertyChangeRegistry = PropertyChangeRegistry()

    override fun addOnPropertyChangedCallback(
            callback: Observable.OnPropertyChangedCallback) {
        callbacks.add(callback)
    }

    override fun removeOnPropertyChangedCallback(
            callback: Observable.OnPropertyChangedCallback) {
        callbacks.remove(callback)
    }

    /**
     * Notifies observers that all properties of this instance have changed.
     */
    fun notifyChange() {
        callbacks.notifyCallbacks(this, 0, null)
    }

    /**
     * Notifies observers that a specific property has changed. The getter for the
     * property that changes should be marked with the @Bindable annotation to
     * generate a field in the BR class to be used as the fieldId parameter.
     *
     * @param fieldId The generated BR id for the Bindable field.
     */
    fun notifyPropertyChanged(fieldId: Int) {
        callbacks.notifyCallbacks(this, fieldId, null)
    }
}

```