## GuessTheWorld 

<img src="./image/image-20200612210604216.png" alt="image-20200612210604216" style="zoom:50%;" />

### UI controller

A *UI controller* is a UI-based class such as `Activity` or `Fragment`. 

A UI controller should **only contain logic that handles UI and operating-system interactions such as displaying views and capturing user input**. 

**Don't put decision-making logic**, such as logic that determines the text to display, into the UI controller.

In the GuessTheWord starter code, the UI controllers are the three fragments: `GameFragment`, `ScoreFragment,` and `TitleFragment`. 

Following the "separation of concerns" design principle, the `GameFragment` is **only responsible for drawing game elements to the screen and knowing when the user taps the buttons, and nothing more**. When the user taps a button, this information is passed to the `GameViewModel`.

### ViewModel

A [`ViewModel`](https://developer.android.com/reference/android/arch/lifecycle/ViewModel) holds data to be displayed in a fragment or activity associated with the `ViewModel`. A `ViewModel` can do simple calculations and transformations on data to prepare the data to be displayed by the UI controller. In this architecture, the `ViewModel` performs the **decision-making**.

The `GameViewModel` holds data like the **score value**, **the list of words**, and **the current word**, because this is the data to be displayed on the screen. The `GameViewModel` also contains the business logic to perform simple calculations to decide what the current state of the data is.

### ViewModelProvider

A [`ViewModelFactory`](https://developer.android.com/reference/android/arch/lifecycle/ViewModelProvider.Factory) instantiates `ViewModel` objects, with or without constructor parameters.

![img](./image/d115344705100cf1.png)

During configuration changes such as screen rotations, UI controllers such as fragments are re-created. However, `ViewModel` instances survive. If you create the `ViewModel` instance using the `ViewModel` class, a new object is created every time the fragment is re-created. Instead, create the `ViewModel` instance using a [`ViewModelProvider`](https://developer.android.com/reference/android/arch/lifecycle/ViewModelProvider).

![img](./image/README.png)

**Important:** Always use [`ViewModelProvider`](https://developer.android.com/reference/android/arch/lifecycle/ViewModelProvider) to create `ViewModel` objects rather than directly instantiating an instance of `ViewModel`.

How `ViewModelProvider` works:

- `ViewModelProvider` returns an existing `ViewModel` if one exists, or it creates a new one if it does not already exist.
- `ViewModelProvider` creates a `ViewModel` instance in association with the given scope (an activity or a fragment).
- The created `ViewModel` is retained as long as the scope is alive. For example, if the scope is a fragment, the `ViewModel` is retained until the fragment is detached.

### Attach observers to the LiveData objects

**The observer pattern**

The *observer pattern* is a software design pattern. It specifies communication between objects: an *observable* (the "subject" of observation) and *observers*. An observable is an object that notifies observers about the changes in its state.

![img](./image/b608df5e5e5fa4f8.png)

In the case of `LiveData` in this app, the observable (subject) is the `LiveData` object, and the observers are the methods in the UI controllers, such as fragments. A state change happens whenever the data wrapped inside `LiveData` changes. The `LiveData` classes are crucial in communicating from the `ViewModel` to the fragment.

**code:**

The observer that you just created receives an event when the data held by the observed `LiveData` object changes. Inside the observer, update the score `TextView` with the new score.

```kotlin
viewModel.score.observe(viewLifecycleOwner, Observer { newScore ->
   binding.scoreText.text = newScore.toString()
})
```

### Encapsulate the LiveData

```kotlin
private val _word = MutableLiveData<String>()
val word: LiveData<String>
   get() = _word
```

### Add ViewModel data binding

**Current app architecture:**

In your app, the views are defined in the XML layout, and the data for those views is held in `ViewModel` objects. Between each view and its corresponding `ViewModel` is a UI controller, which acts as a relay between them.

![img](./image/3f68038d95411119.png)

For example:

- The **Got It** button is defined as a `Button` view in the `game_fragment.xml` layout file.
- When the user taps the **Got It** button, a click listener in the `GameFragment` fragment calls the corresponding click listener in `GameViewModel`.
- The score is updated in the `GameViewModel`.

The `Button` view and the `GameViewModel` don't communicate directly—they need the click listener that's in the `GameFragment`.

**ViewModel passed into the data binding**

It would be simpler if the views in the layout communicated directly with the data in the `ViewModel` objects, without relying on UI controllers as intermediaries.

![img](./image/7f26738df2266dd6.png)

`ViewModel` objects hold all the UI data in the GuessTheWord app. By passing `ViewModel` objects into the data binding, you can automate some of the communication between the views and the `ViewModel` objects.

In this task, you associate the `GameViewModel` and `ScoreViewModel` classes with their corresponding XML layouts. You also set up listener bindings to handle click events.

```kotlin
// Set the viewmodel for databinding - this allows the bound layout access 
// to all the data in the ViewModel
binding.gameViewModel = viewModel
```

Data binding creates a listener and sets the listener on the view. When the listened-for event happens, the listener evaluates the lambda expression. Listener bindings work with the Android Gradle Plugin version 2.0 or higher. To learn more, read [Layouts and binding expressions](https://developer.android.com/topic/libraries/data-binding/expressions#listener_bindings).

### Add transformation for the LiveData

The [`Transformations.map()`](https://developer.android.com/reference/androidx/lifecycle/Transformations.html#map(androidx.lifecycle.LiveData, androidx.arch.core.util.Function)) method provides a way to perform data manipulations on the source `LiveData` and return a result `LiveData` object. These transformations aren't calculated unless an observer is observing the returned `LiveData` object.

This method takes the source `LiveData` and a function as parameters. The function manipulates the source `LiveData`.

**Add hint**

```kotlin
// The Hint for the current word
val wordHint = Transformations.map(word) { word ->
   val randomPosition = (1..word.length).random()
   "Current word has " + word.length + " letters" +
           "\nThe letter at position " + randomPosition + " is " +
           word.get(randomPosition - 1).toUpperCase()
}
```

