# Data Container

{% hint style="info" %}
Although `DataContainer` is advanced and powerful tool, its usage is discouraged in favor of other reactive state containers, like Redux store, StateFlow and other observables.
{% endhint %}

The `kvision-datacontainer` module contains a data component, which allows you to easily display a list of arbitrary items with one or two-way bindings.

### Data model

You have to keep your data in the instance of the `ObservableList` collection. If you like your data elements to be immutable just use a data class.

```kotlin
data class Tweet(val date: Date, 
    val nick: String, 
    val message: String)
```

But sometimes it may be easier to use a mutable objects - in such case you should extend`io.kvision.data.BaseDataComponent` and use delegated properties when declaring your class.

```kotlin
class Tweet(date: Date, nick: String, message: String) : BaseDataComponent() {
    var date by obs(date)
    var nick by obs(nick)
    var message by obs(message)
}
```

To create your data model just use a builder for the `ObservableList`.

```kotlin
import io.kvision.state.observableListOf

val tweets = observableListOf<Tweet>()
```

Of course you may also initialize your list with some data elements.

```kotlin
import io.kvision.state.observableListOf

val tweets = observableListOf(
    Tweet(Date(), "User1", "How are you?"),
    Tweet(Date(), "User2", "Good! Thx!")
)
```

Your data model can be modified by adding and removing elements from the list and by modifying list elements if they are mutable.

```kotlin
tweets.clear()
tweets.add(Tweet(Date(), "User3", "Hello all!"))
tweets.forEach { it.date = Date() } // only for mutable data elements
```

### Data container

You can use `io.kvision.data.DataContainer` class to bind your data model to the elements of the user interface. `DataContainer` is an universal component which allows you to define how your data should be presented. The constructor of the data container class takes three required parameters. First is the `model` \(an `ObservableList` instance\), the second is a `factory` function and the third is a internal container instance \(when using DSL builders the third parameter will default to the `VPanel` instance\). 

The `factory` function does the most of the work - it defines the transformation from a single data element to a user interface component. Its parameters allow you to get access to the data element, the index of the element in the data model list and the model list itself. Of course the component returned by the `factory` function can also be a container with other components. It can be just as complex as your application requires.

```kotlin
data class Data(val text: String)
val model = observableListOf(Data("One"), Data("Two"), Data("Three"))

dataContainer(model, { element, _, _ ->
    Span(element.text)
})
```

{% hint style="info" %}
You can safely use DSL builder functions inside a `factory` function. KVision will  automatically detect if you are using DSL builders or just return a component to be used.
{% endhint %}

If you use mutable data elements, you can easily mutate the element \(e.g. in an event handler\).

```kotlin
class Data(text: String) : BaseDataComponent() {
    var text by obs(text)
}
val model = observableListOf(Data("One"), Data("Two"), Data("Three"))

dataContainer(model, { element, _, _ ->
    Span(element.text).onEvent {
        click = {
            element.text = "Clicked"
        }
    }
})
```

### Internal container

All components created with the `factory` function are put inside an internal container. You can specify its class with the `container` parameter.

```kotlin
data class Data(val text: String)
val model = observableListOf(Data("One"), Data("Two"), Data("Three"))

dataContainer(model, { element, _, _ ->
    Span(element.text)
}, container = HPanel(spacing = 10, wrap = FlexWrap.WRAP))
```

{% hint style="info" %}
Note: with DSL builders the internal container will default to the `VPanel` instance.
{% endhint %}

By default the components are added to the given container with the standard `add` method, but it is possible to change this behavior with `containerAdd` parameter function. One of the parameters of `containerAdd` function is the data element, so the process of adding components may depend on the data.

```kotlin
data class Data(val text: String, val order: Int)
val model = observableListOf(Data("One", 3), Data("Two", 2), Data("Three", 1))

dataContainer(model, { element, _, _ ->
    Span(element.text)
}, container = HPanel(spacing = 10), containerAdd = { component, element ->
    this.add(component, element.order)
})
```

### Data transformations

With additional parameters of the data container class, data model can be filtered and sorted before being presented to the user. The `filter`, `sorter` and `sorterType` parameters allow you to specify functions used to transform the data. 

```kotlin
data class Data(val text: String, val order: Int)
val model = observableListOf(Data("One", 3), Data("Two", 2), Data("Three", 1))

dataContainer(model, { element, index, _ ->
    Span(element.text)
}, container = HPanel(spacing = 10), 
filter = { it.order < 3 }, sorter = { it.order }, sorterType = { SorterType.DESC })
```

{% hint style="info" %}
Note: the `index` parameter of the `factory` function is always taken from the original model list before any transformation are applied.
{% endhint %}

