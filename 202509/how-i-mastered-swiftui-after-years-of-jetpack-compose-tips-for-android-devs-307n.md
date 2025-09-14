---
Title: How I Mastered SwiftUI After Years of Jetpack Compose: Tips for Android Devs
Description: 
Author: Ibtihaj Uddin
Date: 2025-09-14T22:01:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>As an Android developer with years immersed in Jetpack Compose, I thought I had UI development figured out. Compose revolutionized how we build Android apps with its declarative paradigm, composable functions, and reactive state handling. But when I decided to expand my skills to iOS development, SwiftUI presented a whole new world. It was familiar in its declarative nature yet distinct in its Apple-flavored approach. This article shares my journey mastering SwiftUI, focusing on the transition from Compose. I'll highlight specific challenges like layout modifiers and state management, offer practical tips for fellow Android devs, compare key concepts with code snippets, discuss preview tools, walk through a mini-project built in both frameworks, and spotlight tools that accelerated my learning. If you're an Android pro eyeing iOS, these insights will help you navigate the switch smoothly.</p>

<h2>
  
  
  My Background and the Spark to Learn SwiftUI
</h2>

<p>I've been building Android apps since the early days of XML layouts and Activities. When Jetpack Compose launched, it felt like a breath of fresh air. No more wrestling with RecyclerViews or ConstraintLayouts; everything became functions that recompose based on state changes. I mastered Material Design components, animations with AnimatedVisibility, and complex UIs for apps in e-commerce and fitness tracking.</p>

<p>The turning point came last year when a client wanted a cross-platform app. While I could have stuck with Kotlin Multiplatform or Flutter, I saw an opportunity to dive into native iOS. SwiftUI, Apple's declarative UI framework introduced in 2019, seemed like the natural counterpart to Compose. Both are reactive and component-based, but I quickly learned the devil is in the details. My goal wasn't just to build iOS apps but to think like an iOS developer. Over six months, I built several projects, read the docs obsessively, and iterated through trial and error. Now, I comfortably switch between the two, and here's how I got there.</p>

<h2>
  
  
  Understanding the Core Similarities and Differences
</h2>

<p>At first glance, SwiftUI and Jetpack Compose share a declarative DNA. In Compose, you write functions annotated with @Composable that describe the UI. SwiftUI uses structs conforming to the View protocol. Both recompose or redraw when state changes, avoiding manual view updates.<br>
But differences emerge quickly. Compose is built on Kotlin, leveraging coroutines and flows for async work. SwiftUI integrates with Swift's async/await and Combine framework. As an Android dev, I appreciated Compose's flexibility with third-party libraries, while SwiftUI feels more opinionated, tightly coupled with Apple's ecosystem.</p>

<p>Let's compare basics. In Compose, a simple button looks like this:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>kotlin@Composable
fun MyButton(onClick: () -&gt; Unit) {
    Button(onClick = onClick) {
        Text("Click Me")
    }
}
</code></pre>

</div>



<p>In SwiftUI:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>swiftstruct MyButton: View {
    var onClick: () -&gt; Void

    var body: some View {
        Button("Click Me") {
            onClick()
        }
    }
}
</code></pre>

</div>



<p>See the parallels? Both are lightweight and composable. But SwiftUI's body property is mandatory, enforcing a single root view. Compose allows multiple siblings without a wrapper, which can lead to more flexible but sometimes messier hierarchies.</p>

<h2>
  
  
  Challenge 1: Layout Modifiers vs. Composables
</h2>

<p>One of my biggest hurdles was layouts. In Compose, you use composables like Row, Column, and Box for arrangement, with modifiers chaining to tweak padding, size, and alignment. Modifiers are powerful: .padding(16.dp).background(Color.Blue).clickable { }.<br>
SwiftUI flips this with modifiers on views. Instead of dedicated composables for layouts, you apply modifiers like .padding(16), .background(Color.blue), .onTapGesture { }. It's similar, but SwiftUI's stacks (HStack, VStack, ZStack) are the equivalents to Row, Column, Box.<br>
The challenge? Compose modifiers apply in the order you chain them, which can be intuitive but error-prone if you forget the sequence. SwiftUI modifiers also chain, but the order matters less for some (like padding before background), yet it tripped me up initially.<br>
Tip for Android devs: Think of SwiftUI modifiers as Compose modifiers but with more emphasis on the view's environment. For example, to center a text:<br>
Compose:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
kotlinBox(modifier = Modifier.fillMaxSize()) {
    Text("Centered", modifier = Modifier.align(Alignment.Center))
}
</code></pre>

</div>



<p>SwiftUI:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>swiftZStack {
    Text("Centered")
        .frame(maxWidth: .infinity, maxHeight: .infinity, alignment: .center)
}
</code></pre>

</div>



<p>Or more idiomatically:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>swiftText("Centered")
    .frame(maxWidth: .infinity, maxHeight: .infinity)
</code></pre>

</div>



<p>Practice chaining modifiers early. I started by recreating simple Compose layouts in SwiftUI, noting how .frame in SwiftUI combines size and alignment, unlike Compose's separate modifiers.</p>

<p>Another pitfall: Lazy loading. Compose has LazyColumn for efficient lists. SwiftUI's List and LazyVStack do similar, but List adds built-in features like swipe actions, which Compose requires extra work for. I struggled with performance in large lists until I learned to use LazyVStack inside ScrollView for custom behaviors.</p>

<h2>
  
  
  Challenge 2: State Management
</h2>

<p>State is the heart of declarative UIs, and this was where my Compose expertise both helped and hindered. In Compose, @State, remember, and MutableState drive recompositions. For shared state, ViewModels with LiveData or StateFlow.</p>

<p>SwiftUI uses @State for local view state, @Binding for passing mutable state down, and @ObservedObject or @StateObject for observable models. The new <a class="mentioned-user" href="https://dev.to/observable">@observable</a> macro in SwiftUI 5 simplifies this.</p>

<p>Coming from Compose, I overcomplicated things by trying to map ViewModels directly. SwiftUI's environment system (@EnvironmentObject) shares state across views without props drilling, unlike Compose's CompositionLocal.</p>

<p>A common challenge: Lifting state up. In both, you hoist state to avoid unnecessary recompositions, but SwiftUI's property wrappers make it feel magical.<br>
Example: A counter.</p>

<p>Compose:<br>
kotlin@Composable<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>fun Counter() {
    var count by remember { mutableStateOf(0) }
    Button(onClick = { count++ }) {
        Text("Count: $count")
    }
}
</code></pre>

</div>



<p>SwiftUI:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>swiftstruct Counter: View {
    @State private var count = 0

    var body: some View {
        Button("Count: \(count)") {
            count += 1
        }
    }
}
</code></pre>

</div>



<p>Tip: For Android devs, start with @State and @Binding. Avoid @ObservedObject until you need Combine publishers. I wasted time fighting change detection until I realized SwiftUI recomposes on any state change, similar to Compose, but with less control over scopes.</p>

<p>For app-wide state, SwiftUI integrates with SwiftData or Core Data, while Compose often uses Room with ViewModels. My tip: Use @AppStorage for simple persistence, like UserDefaults, which has no direct Compose equivalent without SharedPreferences wrappers.<br>
`</p>

<h2>
  
  
  Previews: Xcode vs. Android Studio
</h2>

<p>Previews were a game-changer in my learning. Android Studio's Compose Preview renders composables instantly, with @Preview annotations for multiple configurations.<br>
Xcode's SwiftUI Preview is similar but more integrated. You wrap views in #Preview, and it updates live as you type. No need for separate preview functions per variant; use PreviewProvider for multiples.<br>
Challenge: Xcode previews can be finicky with data dependencies. If your view needs a model, you mock it in the preview. Android Studio handles this well too, but Xcode's canvas lets you interact with the preview, like tapping buttons, which Compose previews don't support natively.<br>
Tip: Leverage Xcode's live previews to iterate faster. I set up a split view: code on left, preview on right. For complex states, use .previewDevice to test on different iPhones. Compared to Android Studio's slower build times for previews, Xcode felt snappier, boosting my productivity.<br>
One downside: Xcode lacks Compose's inspection tools out of the box, but that's where third-party tools shine (more on that later).</p>

<h2>
  
  
  Mini-Project: Building a To-Do List App in Both Frameworks
</h2>

<p>To solidify my understanding, I built a simple to-do list app in both Compose and SwiftUI. This mini-project highlighted transitions in practice. Let's break it down.<br>
First, the requirements: Display a list of tasks, add new ones via text field and button, mark as done with checkboxes, and persist data.<br>
In Jetpack Compose:<br>
I used a ViewModel with a StateFlow&gt;. The UI:<br>
kotlin</p>

<p><code></code>`<br>
data class Task(val id: Int, val text: String, var done: Boolean)</p>

<p>class TodoViewModel : ViewModel() {<br>
    private val _tasks = MutableStateFlow(listOf())<br>
    val tasks: StateFlow&gt; = _tasks.asStateFlow()</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>fun addTask(text: String) {
    _tasks.value = _tasks.value + Task(Random.nextInt(), text, false)
}

fun toggleDone(id: Int) {
    _tasks.value = _tasks.value.map { if (it.id == id) it.copy(done = !it.done) else it }
}
</code></pre>

</div>

<p>}</p>

<p>@Composable<br>
fun TodoScreen(viewModel: TodoViewModel = viewModel()) {<br>
    val tasks by viewModel.tasks.collectAsState()<br>
    var newTask by remember { mutableStateOf("") }</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Column {
    TextField(value = newTask, onValueChange = { newTask = it })
    Button(onClick = { viewModel.addTask(newTask); newTask = "" }) {
        Text("Add")
    }
    LazyColumn {
        items(tasks) { task -&gt;
            Row {
                Checkbox(checked = task.done, onCheckedChange = { viewModel.toggleDone(task.id) })
                Text(task.text)
            }
        }
    }
}
</code></pre>

</div>

<p>}<br>
`<code></code></p>

<p>For persistence, I'd integrate Room, but for simplicity, it's in-memory.<br>
Now, in SwiftUI:<br>
I used an <a class="mentioned-user" href="https://dev.to/observable">@observable</a> class for the model.</p>

<p>swift@Observable<br>
class TodoViewModel {<br>
    var tasks: [Task] = []</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>func addTask(text: String) {
    tasks.append(Task(id: UUID(), text: text, done: false))
}

func toggleDone(id: UUID) {
    if let index = tasks.firstIndex(where: { $0.id == id }) {
        tasks[index].done.toggle()
    }
}
</code></pre>

</div>

<p>}</p>

<p>struct Task: Identifiable {<br>
    let id: UUID<br>
    let text: String<br>
    var done: Bool<br>
}</p>

<p>struct TodoView: View {<br>
    @State private var newTask = ""<br>
    @StateObject private var viewModel = TodoViewModel()</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>var body: some View {
    VStack {
        TextField("New Task", text: $newTask)
        Button("Add") {
            viewModel.addTask(text: newTask)
            newTask = ""
        }
        List {
            ForEach(viewModel.tasks) { task in
                HStack {
                    Toggle(isOn: Binding(
                        get: { task.done },
                        set: { viewModel.toggleDone(id: task.id) }
                    )) {
                        Text(task.text)
                    }
                    .toggleStyle(.checkbox)
                }
            }
        }
    }
}
</code></pre>

</div>

<p>}</p>

<p>Key insights from building this:</p>

<ul>
<li>Lists: Compose's LazyColumn is flexible for custom items; SwiftUI's List adds sectioning and editing for free.</li>
<li>State binding: SwiftUI's $newTask binds directly, like Compose's two-way data binding in TextField.</li>
<li>Toggling: Compose uses onCheckedChange; SwiftUI's Toggle with Binding feels more declarative.</li>
</ul>

<p>Challenges: Handling IDs. Compose uses indices or generated IDs; SwiftUI requires Identifiable for ForEach, enforcing uniqueness.<br>
For persistence, SwiftUI pairs nicely with SwiftData:<br>
swift</p>

<p>@Model<br>
class Task {<br>
    @Attribute(.unique) var id: UUID<br>
    var text: String<br>
    var done: Bool<br>
}</p>

<p>Then query with @Query. In Compose, it's a Repository with Room DAO.<br>
This project took me a weekend per framework initially, but now I can prototype in hours. Tip: Build the same app in both to spot patterns. Start with UI, add state, then persistence.</p>

<h2>
  
  
  Tools That Eased My Learning Curve
</h2>

<p>Beyond basics, tools were crucial. SwiftUI Inspector, a free Xcode extension, lets you inspect view hierarchies live, like Compose's Layout Inspector but more seamless. It shows modifiers, frames, and lets you tweak them on the fly.<br>
I also used Reveal app for deeper inspections, visualizing layers and animations. For debugging, SwiftUI's .debugPrint modifier logs view trees, akin to Compose's recomposition logging.<br>
Other gems: Point-Free's SwiftUI tutorials for advanced state management, and Hacking with Swift for basics. Compared to Compose's official docs, Apple's are more visual with sample code.<br>
For previews, I customized with .previewLayout(.sizeThatFits) to focus on components.<br>
Tip: Install SwiftUI Inspector early. It demystified why a modifier wasn't applying, saving hours.</p>

<h2>
  
  
  Advanced Tips: Animations, Navigation, and Performance
</h2>

<p>Once basics clicked, I tackled animations. Compose's animate*AsState is straightforward; SwiftUI's .animation(.default) applies to state changes automatically.<br>
Challenge: Implicit animations in SwiftUI can surprise you, animating everything. Use .animation(nil) to opt out.<br>
Navigation: Compose's NavController with destinations; SwiftUI's NavigationStack with paths. Both declarative, but SwiftUI's is simpler for deep links.<br>
Performance: Both optimize recompositions, but profile with Instruments in Xcode vs. Profiler in Android Studio.<br>
Tip: For Android devs, learn SwiftUI's GeometryReader for responsive layouts, like Compose's LayoutModifier.</p>

<h2>
  
  
  Overcoming Mental Blocks
</h2>

<p>As an Android vet, I resisted Apple's conventions like Swift's optionals and enums. But embracing them made SwiftUI shine. I joined communities like Reddit's r/SwiftUI and Stack Overflow for quick answers.</p>

<h2>
  
  
  Conclusion: Your Path Forward
</h2>

<p>Mastering <a href="https://khancodes.com/blog/category/swift/" rel="noopener noreferrer">SwiftUI</a> after Compose wasn't overnight, but the parallels accelerated it. Focus on modifiers, state, and previews first. Build that to-do app, experiment with tools, and iterate. You'll not only build iOS apps but think cross-platform. With these tips, you'll transition faster than I did. Dive in the water's fine.</p>

